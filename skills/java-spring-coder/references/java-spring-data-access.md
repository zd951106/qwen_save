# Java/Spring 数据访问与事务专题

本文件收录数据访问、对象转换、数据库设计、批量处理、事务和空安全等按需专题规则。

## 1. 对象转换

- ❌ 禁止直接返回 Entity，必须转换为 DTO/Rsp
- 优先使用 MapStruct 等性能更高的对象转换工具；如果系统没有，再考虑 `BeanUtils.copyProperties(source, target)`
- 注意：`BeanUtils.copyProperties` 是浅拷贝，嵌套对象需手动处理

## 2. 数据库表设计规范

### 基础字段要求

实体类继承 `BaseDO` 时，建表语句**必须包含**以下基础字段：

| 字段名 | 类型 | 说明 | 必须 |
|--------|------|------|------|
| `id` | `bigint` | 主键，自增 | ✅ |
| `reserved1` | `int` | 保留字段1 | 否 |
| `create_date_time` | `datetime(3)` | 创建时间 | ✅ |
| `update_date_time` | `datetime(3)` | 更新时间 | ✅ |
| `creator` | `varchar(32)` | 创建者 | ✅ |
| `updater` | `varchar(32)` | 更新者 | ✅ |
| `logic_delete_flag` | `bigint` | 逻辑状态:1-正常,其它-已删除 | ✅ |

### 建表语句模板

```sql
CREATE TABLE `table_name` (
    `id` bigint NOT NULL AUTO_INCREMENT COMMENT '主键',
    -- 业务字段...
    -- 基础字段...
    `reserved1` int DEFAULT 0 COMMENT '保留字段1',
    `creator` varchar(32) DEFAULT '' COMMENT '创建者',
    `create_date_time` datetime(3) NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
    `updater` varchar(32) DEFAULT '' COMMENT '更新者',
    `update_date_time` datetime(3) NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新时间',
    `logic_delete_flag` bigint NOT NULL DEFAULT '1' COMMENT '逻辑状态:1-正常,其它-已删除',
    PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='表注释';
```

### 字段命名规范

| 规则 | 示例 |
|------|------|
| 表名、字段名使用 `snake_case` | `promotion_plan`、`create_date_time` |
| 布尔字段使用 `_flag` 后缀
| 时间字段使用 `date_time` 后缀 | `create_date_time`、`effect_time` |
| 状态字段使用 `status` | `order_status`、`approval_status` |

### 索引规范

| 类型 | 命名规则 | 示例 |
|------|----------|------|
| 主键 | `PRIMARY KEY` | `PRIMARY KEY (id)` |
| 唯一索引 | `uk_表名_字段` | `uk_order_order_no` |
| 普通索引 | `idx_表名_字段` | `idx_order_user_id` |
| 联合索引 | `idx_表名_字段1_字段2` | `idx_order_user_id_status` |

```sql
-- 索引示例
CREATE UNIQUE INDEX `uk_order_order_no` ON `order` (`order_no`);
CREATE INDEX `idx_order_user_id` ON `order` (`user_id`);
CREATE INDEX `idx_order_create_date_time` ON `order` (`create_date_time`);
```

## 3. 批量处理

超过 1000 条记录必须分批处理：

```java
int batchSize = 1000;
for (int i = 0; i < list.size(); i += batchSize) {
    List<Entity> batch = list.subList(i, Math.min(i + batchSize, list.size()));
    saveBatch(batch);
}
```

## 4. 事务控制

### 基本规则

- 多表操作必须加 `@Transactional(rollbackFor = Exception.class)`
- ⚠️ 事务方法必须是 `public`（private 不生效）
- ⚠️ 避免同类方法内部调用（代理失效）
- 长事务拆分，避免锁表时间过长
- ❌ 禁止在 `@Transactional` 方法中进行远程 RPC 调用、复杂的外部 API 请求或大文件 IO。先执行外部请求，拿到结果后再进入 `@Transactional` 方法操作数据库。或使用 `TransactionTemplate` 编程事务
- 尽量使用手动事务，避免 `@Transactional` 方法中包含复杂的逻辑。

### 多数据源事务限制

**⚠️ 重要**：`@Transactional` 只对主数据源生效，事务方法中 ❌ 禁止混用多个数据源。

```java
// ❌ 错误：事务中混用 MySQL 和 Doris
@Transactional(rollbackFor = Exception.class)
public void syncData() {
    // Doris 查询不在事务管理范围内
    List<Data> dorisData = dorisMapper.selectList();
    // MySQL 写入在事务中
    mysqlMapper.saveBatch(dorisData);
}

// ✅ 正确：拆分方法，事务只包裹单数据源操作
public void syncData() {
    // 1. 非事务方法查询 Doris
    List<Data> dorisData = queryFromDoris();
    // 2. 事务方法写入 MySQL
    saveToMysql(dorisData);
}

@Transactional(rollbackFor = Exception.class)
public void saveToMysql(List<Data> data) {
    mysqlMapper.saveBatch(data);
}
```

### 同类调用代理失效

```java
// ❌ 同类调用事务不生效
public void methodA() {
    this.methodB();  // methodB 的 @Transactional 不生效
}

// ✅ 正确：注入自身或拆分到另一个 Service
@Autowired
private ProductService self;

public void methodA() {
    self.methodB();  // 通过代理调用，事务生效
}
```

## 5. 空安全处理

```java
// 集合判空
if (CollectionUtils.isEmpty(list)) {
    return Collections.emptyList();
}

// 链式调用防空
String city = Optional.ofNullable(user)
    .map(User::getAddress)
    .map(Address::getCity)
    .orElse("");
```
