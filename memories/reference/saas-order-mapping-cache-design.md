---
name: SaaS订单映射缓存改造设计
description: SaasOrderMappingService缓存改造完整设计——业务背景、缓存策略、读写流程、并发安全、防穿透、线程池隔离、踩坑记录，可作为"高频读低频写"场景的通用缓存设计参考模板
type: reference
---

# SaaS 交易映射缓存改造设计

## 一、业务背景

### 1.1 场景描述

OpenSaaS 支付网关中，每次交易（支付/退款/查询/关单/异步通知）都需要通过"发起方编号 + 订单号"或"发起方编号 + 商户订单号"查询映射表获取商户号，用于路由到正确的支付通道。

### 1.2 核心问题

- **两张映射表**：`t_saas_order_mapping`（按订单号）和 `t_saas_mer_order_mapping`（按商户订单号）
- **高频读**：每次 OpenSaaS 交易都查，QPS 高时 DB 成为瓶颈
- **低频写**：仅在支付/退款时写入，频率远低于读
- **数据一致性要求不高**：允许短暂不一致，最终一致即可

### 1.3 调用链路

```
Controller → Adapter → SaasOrderMappingService.resolveMapping()
                                              └→ cache-aside (Redis → DB → 回写)
Controller → BaseService → SaasOrderMappingService.saveMapping()
                                              └→ DB insert → afterCommit → 异步写缓存
```

---

## 二、缓存方案设计

### 2.1 缓存策略：Cache-Aside + SETNX

| 操作 | 缓存行为 | 说明 |
|------|---------|------|
| **读** | Redis miss → DB → SETNX 回写 Redis（10s TTL） | 读路径用 SETNX，已有值不覆盖 |
| **读空值** | DB 未命中 → SETNX 写 NULL_CACHE 标记（10s TTL） | 防缓存穿透 |
| **写（saveMapping）** | afterCommit → 异步 SET 写缓存 | 写路径权威，直接 SET 覆盖 |
| **更新（updateMapping）** | 更新 DB → afterCommit → 异步 SET 写缓存 | 同上 |

### 2.2 双 Key 设计

两个维度分别缓存，覆盖两种查询场景：

```
# 按订单号查询
Key: posxSaasOrderMapping{organizNo}{orderId}
Value: {"organizNo":"xxx","merchantNo":"xxx","merchantOrderId":"xxx","orderId":"xxx","extendInfo":"xxx"}

# 按商户订单号查询
Key: posxSaasOrderMapping{organizNo}{merOrderId}
Value: 同上
```

### 2.3 TTL 设计

- **数据缓存 TTL**：10s（注释写 40 分钟但代码实际为 10s）
- **空值缓存 TTL**：10s
- **设计考量**：
  - 短 TTL 适配"允许短暂不一致"的业务特性
  - 10s 足够覆盖绝大多数重复查询
  - TTL 越短，数据不一致窗口越小，但 DB 穿透越多
  - **优化建议**：可适当增加到 30~60s，或根据业务 QPS 调优

### 2.4 缓存穿透防护

```java
// DB 未命中时写入空值标记，防止恶意/高频查询不存在的 key 穿透 DB
private void setNullCache(String cacheKey) {
    redisTemplate.opsForValue()
        .setIfAbsent(cacheKey, "NULL_CACHE", 10, TimeUnit.SECONDS);
}
```

### 2.5 缓存异常降级

```java
// 缓存读取异常时，删除异常 key 并回退到 miss，走 DB 查询
try {
    String json = redisTemplate.opsForValue().get(cacheKey);
    // ...
} catch (Exception e) {
    log.warn("读取缓存异常, cacheKey={}", cacheKey, e);
    redisTemplate.delete(cacheKey);  // 清除异常数据
    return CacheLookupResult.miss(); // 降级到 DB
}
```

**原则**：缓存永远不应成为故障点，异常时自动降级到 DB。

---

## 三、并发安全设计

### 3.1 SETNX 语义

```
读路径：SETNX（已有值不覆盖）
写路径：SET（直接覆盖，因为写路径是权威来源）
```

**设计逻辑**：
- 读路径：目标是把值塞进缓存，但如果有其他请求已经塞了，就以它为准（避免覆盖）
- 写路径：DB 已写入，缓存必须反映最新状态，所以直接 SET 覆盖

### 3.2 线程池隔离

```java
private ThreadPoolExecutor cacheWriteExecutor;

@PostConstruct
public void init() {
    cacheWriteExecutor = new ThreadPoolExecutor(
        corePoolSize,       // 默认 4，可配置
        maxPoolSize,        // = corePoolSize（固定大小）
        60L, TimeUnit.SECONDS,
        new LinkedBlockingQueue<>(queueCapacity),  // 默认 1000，有界队列
        daemon thread factory,
        new DiscardPolicy()  // 队列满直接丢弃，不阻塞业务线程
    );
}
```

**关键设计决策**：
- **有界队列 + DiscardPolicy**：绝不阻塞业务线程（afterCommit 回调节点）
- **守护线程**：不阻止 JVM 退出
- **固定线程数**：避免无限制创建线程
- **可配置化**：`sass.order.mapping.cache.write.pool.size` / `sass.order.mapping.cache.write.queue.capacity`

### 3.3 事务边界

```java
// 写路径：必须在事务提交后才能写缓存
TransactionSynchronizationManager.registerSynchronization(new TransactionSynchronization() {
    @Override
    public void afterCommit() {
        cacheWriteExecutor.submit(() -> doWriteCache(...));
    }
});
```

**关键原则**：**永远在 afterCommit 写缓存，不在事务内写**。事务可能回滚，提前写会导致缓存脏数据。

---

## 四、数据模型

### 4.1 表结构

```sql
-- 订单号映射表
t_saas_order_mapping (
    primary_key BIGINT,          -- 主键
    organiz_no VARCHAR,          -- 发起方编号
    order_id VARCHAR,            -- 系统订单号
    merchant_order_id VARCHAR,   -- 商户订单号
    merchant_no VARCHAR,         -- 商户号（核心查询字段）
    extend_info VARCHAR,         -- 扩展信息（JSON）
    create_date_time TIMESTAMP,
    update_date_time TIMESTAMP,
    logic_delete_flag TINYINT,   -- 0正常 1删除
    reserved1 INT
)

-- 商户订单号映射表（字段完全一致）
t_saas_mer_order_mapping (...)
```

### 4.2 缓存 JSON 结构

```json
{
    "organizNo": "发起方编号",
    "merchantNo": "商户号",
    "merchantOrderId": "商户订单号",
    "orderId": "系统订单号",
    "extendInfo": "扩展信息"
}
```

**为什么用 JSONObject 而非直接序列化 Entity？**
- 显式字段控制：避免 Entity 中的 `createDateTime`、`updateDateTime`、`logicDeleteFlag` 等非业务字段进入缓存
- 体积更小：减少 Redis 内存占用

---

## 五、数据一致性分析

### 5.1 不一致窗口全景

| 时间点 | 事件 | DB | Redis | 不一致？ |
|--------|------|-----|-------|----------|
| T0 | 初始状态 | merchantNo=A | (无) | - |
| T1 | 请求1读取 | A | SETNX A(TTL=10s) | ❌ 一致 |
| T2 | updateMapping(A→B) | B | A（SETNX 不覆盖） | ⚠️ 不一致 |
| T3 | 任意读取 | B | A（读到旧值） | ⚠️ 不一致 |
| T4 | 缓存过期 | B | (无) | ❌ 一致（下次读回源） |
| T5 | 请求读取 | B | SET B | ❌ 一致 |

**结论**：
- 最大不一致窗口 = TTL（10s）
- 因为写路径走 SET（覆盖），afterCommit 异步写入后会覆盖旧值
- 但 afterCommit 和执行之间有时间差（线程池调度 + 网络），不一致窗口取决于异步执行延迟

### 5.2 并发写入场景

```
线程A: saveMapping → DB insert → afterCommit → 异步 SET key(A)
线程B: updateMapping → DB update(A→B) → afterCommit → 异步 SET key(B)

可能的执行顺序：
1. DB insert A
2. DB update A→B
3. 异步 SET key(A)   ← 覆盖了 B！
4. 异步 SET key(B)
```

**缓解**：afterCommit 的执行顺序不保证与事务提交顺序一致，但通过 SET 覆盖 + 短 TTL 控制影响范围。

**改进方向**（如果一致性要求更高）：
- Redis DEL + SET（删除后写入，而非 SET 覆盖）
- 延迟双删
- 版本号/时间戳比较后写入

---

## 六、性能与容量评估

### 6.1 缓存命中率估算

- 10s TTL：对于同一 orderId/merOrderId 在 10s 内的重复查询全部命中
- 实际命中率取决于业务场景：
  - 支付场景：prepay → pay → query 链路在秒级内完成，命中率极高
  - 退款场景：可能间隔更长，命中率较低
  - 查询/关单：单次查询，缓存收益有限

### 6.2 线程池容量

- 默认 core=4, queue=1000
- 高峰时队列满 → DiscardPolicy 丢弃，业务无损（下次走 DB）

### 6.3 缓存废弃

- 如果需要主动废弃缓存，当前实现无此能力
- 依赖 TTL 自动过期

---

## 七、踩坑与注意事项

### 7.1 读/写 SETNX 不对称问题

| 路径 | 方法 | 说明 |
|------|------|------|
| resolveMapping（读） | SETNX | 已有值不覆盖 |
| doWriteCache（写） | SET | 直接覆盖 |

这是**有意设计**（写路径权威），但容易让人困惑。如果误解为"全都用 SETNX 防止旧数据覆盖新数据"，实际写路径是用 SET 强制覆盖。

### 7.2 afterCommit 与线程池

- `TransactionSynchronization.afterCommit()` 在事务提交后执行
- 如果线程池满（queue+core 全满），DiscardPolicy 静默丢弃
- 业务无感知，下次查询走 DB
- **监控建议**：增加线程池队列长度监控指标，便于发现队列积压

### 7.3 TTL 实际值与注释不符

`CommonConstant.java` 注释写"过期时间与 t_order 一致 40 分钟"，但代码中 `CACHE_TTL_SECONDS = 10`。可能是历史遗留问题。

> **2026-07-15**：经讨论确认，10s TTL 是为了快速数据一致，后续可能调整到更长。

### 7.4 拆分两张表 vs 一张表

当前 `t_saas_order_mapping` 和 `t_saas_mer_order_mapping` 是两张独立表，但字段完全一致。设计原因是：
- 分表后各自用 `organizNo + orderId` 和 `organizNo + merchantOrderId` 作为查询条件
- 避免一张表上两个不同的查询维度导致索引膨胀
- 但带来了双写的事务一致性问题（已在同一事务中处理）

---

## 八、通用缓存设计 Checklist

适用场景：**高频读 + 低频写 + 允许短暂不一致**

| 检查项 | 当前设计 | 通用建议 |
|--------|---------|---------|
| 缓存策略 | Cache-Aside + SETNX | Cache-Aside 是通用默认选择 |
| 缓存穿透 | NULL_CACHE 空值标记 | 必须：防止恶意/不存在 key 穿透 |
| 缓存雪崩 | 短 TTL 分散过期 | 增加：随机偏移 TTL，避免集中过期 |
| 并发安全 | SETNX（读）+ SET（写） | 根据一致性要求选择策略 |
| 事务边界 | afterCommit 写缓存 | 必须：不在事务内写缓存 |
| 线程池隔离 | 专门线程池 | 建议：异步写缓存独立线程池 |
| 降级策略 | 缓存异常 → DB | 必须：缓存不可用时自动降级 |
| 可观测性 | 日志记录 | 建议：增加命中率/队列长度监控 |
| TTL 设置 | 10s | 根据业务允许的不一致窗口设定 |
| 缓存 Key 规范 | `业务前缀+主键1+主键2` | 统一命名规范，便于运维 |
| 缓存 Value 结构 | 显式字段 JSONObject | 控制缓存体积，避免序列化非业务字段 |

---

## 九、改造前后对比

| 维度 | 改造前 | 改造后 |
|------|--------|--------|
| 读路径 | 每次查 DB | Redis 命中时 0 次 DB |
| 写路径 | DB insert × 2 | DB insert × 2 + 异步缓存 |
| 穿透防护 | 无 | NULL_CACHE 标记 |
| 降级能力 | 无（DB 故障即失败） | 缓存异常自动降级 DB |
| 并发安全 | DB 行锁 | DB 行锁 + SETNX |
| 可观测性 | 无 | 日志 + 可扩展监控 |
