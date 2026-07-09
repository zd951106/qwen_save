# Java/Spring 运行期专题

本文件收录并发、缓存、配置安全、异步、消息事务、安全与性能等按需专题规则。

## 1. 并发控制

### 乐观锁（推荐）

适用于冲突较少的场景：

```java
// Entity 增加版本号
@Version
private Integer version;
```

更新时检查版本号，失败则抛出异常或重试。

### 分布式锁

使用 Redisson `RLock`，如果项目中有封装类则优先使用封装工具：

- 设置合理的等待时间和锁超时时间
- 在 finally 中释放锁
- 只释放自己持有的锁 `lock.isHeldByCurrentThread()`
- 注意看门狗自动续期逻辑，确保业务耗时不超时。

## 2. 缓存规范

### Key 命名

格式：`{业务}:{模块}:{标识}`

```java
String key = "promotion:plan:" + planId;
String key = "user:info:" + userId;
```

### TTL 设置

| 数据类型 | TTL | 说明 |
|---------|-----|-----|
| 热点数据 | 1-5 分钟 | 高频访问 |
| 普通数据 | 30 分钟 | 一般业务数据 |
| 配置数据 | 1 小时 | 变更少 |
| 永不过期 | ❌ **禁止** | 必须设置 TTL |

### 缓存穿透防护

空值也缓存（短 TTL 如 5 分钟），防止缓存穿透。

### 缓存更新策略

先更新数据库，再删除缓存。

## 3. 配置安全

以下配置 ❌ 禁止写在代码或本地配置文件：

- 数据库密码
- Redis 密码
- 第三方 API Key / Secret
- JWT 密钥
- 加密盐值

✅ 敏感配置存储在 Nacos 配置中心，按环境隔离。

- ❌ 禁止在日志中打印敏感配置。

## 4. 异步处理

- 使用 `@Async("线程池名")` 指定线程池
- ❌ 禁止使用默认 `SimpleAsyncTaskExecutor`
- ❌ 禁止在同类中调用 `@Async` 方法（代理失效）
- ❌ 禁止在 `@Async` 方法中使用 `@Transactional`（事务不生效）

异步方法需要返回值时使用 `CompletableFuture<T>`。

## 5. 分布式事务（RocketMQ 事务消息）

项目使用 RocketMQ 事务消息实现最终一致性：

1. 发送半消息（Half Message）→ MQ 暂存，不投递
2. 执行本地事务
3. 提交/回滚 → MQ 决定投递或丢弃
4. 异常时 MQ 回查本地事务状态

### 关键点

- 使用 `@RocketMQTransactionListener(txProducerGroup = "xxx")` 实现事务监听器
- `executeLocalTransaction`：执行本地事务，返回 COMMIT/ROLLBACK/UNKNOWN
- `checkLocalTransaction`：事务回查，必须实现幂等查询
- 消费端也要做幂等（Redis 去重）

### 回查幂等

```java
@Override
public RocketMQLocalTransactionState checkLocalTransaction(Message msg) {
    String bizId = (String) msg.getHeaders().get("bizId");
    boolean exists = orderMapper.existsByBizId(bizId);
    return exists ? COMMIT : ROLLBACK;
}
```

## 6. 安全规范

### XSS 防护

- 全局 XssFilter 过滤请求参数
- 富文本字段使用 Jsoup 白名单清洗

## 7. 性能优化

### N+1 查询

❌ 禁止循环查询数据库：

```java
// ❌ 错误
for (Order order : orders) {
    User user = userMapper.selectById(order.getUserId());  // N 次查询
}

// ✅ 正确：批量查询 + 内存关联
Set<Long> userIds = orders.stream().map(Order::getUserId).collect(Collectors.toSet());
Map<Long, User> userMap = userMapper.selectBatchIds(userIds).stream()
    .collect(Collectors.toMap(User::getId, u -> u));
```

### 深度分页

❌ 深度分页禁止使用 OFFSET（性能差）：

```sql
-- ❌ 错误
SELECT * FROM product LIMIT 100000, 10;

-- ✅ 正确：游标分页
SELECT * FROM product WHERE id > #{lastId} ORDER BY id LIMIT 10;
```

### 大数据量导出

使用流式查询 `@Options(fetchSize = 1000)` + 逐条写入，避免 OOM。

## 8. 设计模式

| 模式 | 适用场景 | 说明 |
|------|----------|------|
| 策略模式 | 多分支业务逻辑 | 如支付方式、导出格式 |
| 模板方法 | 流程固定、步骤可变 | 如导出流程 |
| 责任链 | 多级审批、多步校验 | 如审批流 |
