# Java/Spring 质量与交付专题

本文件收录接口文档、测试规范与代码评审清单等按需专题规则。

## 1. 接口文档（Apifox）

项目使用 Apifox IDEA 插件，通过 Javadoc 注释自动生成接口文档。

### 注释规范

```java
/**
 * 商品管理
 */
@RestController
public class ProductController {

    /**
     * 分页查询商品
     * @param req 查询条件
     * @return 商品分页列表
     */
    @PostMapping("/page")
    public CommonResult<IPage<ProductDTO>> getPage(@Valid @RequestBody ProductPageReq req) { }
}
```

### 字段注释

```java
/**
 * 商品名称（模糊匹配）
 * @mock 阿莫西林
 */
private String name;

/**
 * 状态：0-下架，1-上架
 * @mock 1
 */
private Integer status;
```

### 特殊标签

| 标签 | 用途 |
|------|------|
| `@mock` | 字段示例值 |
| `@ignore` | 忽略该字段，不生成文档 |

## 2. 测试规范

| 类型 | 命名 | 说明 | 是否必须 |
|-----|------|-----|---------|
| Mock 测试 | `*Test.java` | 隔离外部依赖，纯逻辑测试 | **必须** |
| 集成测试 | `*IT.java` | 真实环境测试，需数据库 | 视环境 |

### Mock 测试必须覆盖

- 正常流程（happy path）
- 边界条件（空值、临界值）
- 异常分支（参数错误、业务异常）
- 依赖返回异常

### 命名规范

格式：`方法名_场景`

```java
void getById_success()
void getById_notFound()
void getById_nullId()
```

## 3. 代码评审 Checklist

### 必查项

| 检查点 | 说明 |
|--------|------|
| 命名规范 | 类名、方法名、变量名是否符合规范 |
| 日志规范 | 是否有业务标识，是否使用占位符 |
| 异常处理 | 是否使用统一异常，是否有兜底处理 |
| 参数校验 | Controller 是否有 @Valid，Service 是否有业务校验 |
| SQL 安全 | 是否使用 #{}，是否有 SQL 注入风险 |
| 事务边界 | 多表操作是否有事务，是否混用数据源 |
| 空指针 | 是否有 NPE 风险，集合是否判空 |
| 幂等性 | 写接口是否保证幂等 |

### 性能检查

| 检查点 | 说明 |
|--------|------|
| N+1 查询 | 是否有循环查询数据库 |
| 深度分页 | 是否使用游标分页 |
| 批量处理 | 超过 1000 条是否分批 |
