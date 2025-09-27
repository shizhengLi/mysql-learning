# MySQL性能优化实战（三）：查询优化与索引策略

## 前言

在前两篇文章中，我们深入了解了MySQL的基础架构和InnoDB的高级特性。本篇将聚焦于MySQL性能优化的实战技巧，帮助你从源码层面理解查询优化器的工作原理，并掌握实际的性能调优方法。

## 1. MySQL查询优化器架构

### 1.1 查询优化器的整体流程

```cpp
// 查询优化的主要步骤
int optimize_query(THD *thd, TABLE_LIST *tables) {
    // 1. 查询重写
    query_rewrite(thd);

    // 2. 生成逻辑查询计划
    logical_plan = generate_logical_plan(tables);

    // 3. 物理查询优化
    physical_plan = optimize_physical_plan(logical_plan);

    // 4. 成本估算
    cost = estimate_cost(physical_plan);

    // 5. 选择最优执行计划
    best_plan = select_best_plan(physical_plans);

    return best_plan;
}
```

### 1.2 成本模型详解

```cpp
// 成本计算模型
struct cost_model_t {
    double io_cost;         // I/O成本
    double cpu_cost;        // CPU成本
    double memory_cost;     // 内存成本
    double network_cost;    // 网络成本

    double total_cost() {
        return io_cost + cpu_cost + memory_cost + network_cost;
    }
};
```

## 2. 执行计划深度解析

### 2.1 EXPLAIN命令详解

```sql
-- 基础执行计划
EXPLAIN SELECT * FROM users WHERE id = 1;

-- 格式化输出
EXPLAIN FORMAT=JSON SELECT * FROM users WHERE id = 1;

-- 包含执行信息
EXPLAIN ANALYZE SELECT * FROM users WHERE id = 1;
```

### 2.2 执行计划关键字段解析

```json
{
  "query_block": {
    "select_id": 1,
    "cost_info": {
      "query_cost": "1.00"
    },
    "table": {
      "table_name": "users",
      "access_type": "const",
      "possible_keys": ["PRIMARY"],
      "key": "PRIMARY",
      "key_length": "4",
      "used_key_parts": ["id"],
      "ref": ["const"],
      "rows_examined_per_scan": 1,
      "rows_produced_per_join": 1,
      "filtered": "100.00",
      "cost_info": {
        "read_cost": "0.00",
        "eval_cost": "1.00",
        "prefix_cost": "1.00",
        "data_read_per_join": "40"
      },
      "used_columns": ["id", "username", "email", "created_at"]
    }
  }
}
```

### 2.3 访问类型详解

| 访问类型 | 描述 | 性能 |
|----------|------|------|
| const    | 主键/唯一索引查找 | 最优 |
| eq_ref   | 主键/唯一索引连接 | 优秀 |
| ref      | 普通索引查找 | 良好 |
| range    | 索引范围扫描 | 一般 |
| index    | 索引全扫描 | 较差 |
| ALL      | 全表扫描 | 最差 |

## 3. 索引优化策略

### 3.1 索引选择原则

```cpp
// 索引选择算法
bool should_use_index(TABLE *table, KEY *key, const Item *cond) {
    // 1. 选择性评估
    double selectivity = calculate_selectivity(key, cond);

    // 2. 成本估算
    double index_cost = estimate_index_access_cost(key, cond);
    double table_scan_cost = estimate_table_scan_cost(table);

    // 3. 选择更优的访问方式
    return index_cost < table_scan_cost;
}
```

### 3.2 最佳索引设计

#### 3.2.1 单列索引

```sql
-- 创建单列索引
CREATE INDEX idx_username ON users(username);

-- 查询优化器选择索引
SELECT * FROM users WHERE username = 'zhangsan';
```

#### 3.2.2 复合索引

```sql
-- 创建复合索引（最左前缀原则）
CREATE INDEX idx_name_email ON users(username, email);

-- 有效使用索引
SELECT * FROM users WHERE username = 'zhangsan' AND email = 'test@example.com';
SELECT * FROM users WHERE username = 'zhangsan';

-- 无效使用索引（不遵循最左前缀）
SELECT * FROM users WHERE email = 'test@example.com';
```

#### 3.2.3 覆盖索引

```sql
-- 创建覆盖索引
CREATE INDEX idx_covering ON users(username, email, created_at);

-- 使用覆盖索引（避免回表）
SELECT username, email, created_at FROM users WHERE username = 'zhangsan';
```

### 3.3 索引失效场景

```sql
-- 1. 使用函数导致索引失效
SELECT * FROM users WHERE UPPER(username) = 'ZHANGSAN';

-- 2. 类型转换导致索引失效
SELECT * FROM users WHERE username = 123;  -- username是字符串类型

-- 3. 使用NOT、!=、<>等操作符
SELECT * FROM users WHERE username != 'zhangsan';

-- 4. LIKE以通配符开头
SELECT * FROM users WHERE username LIKE '%zhang';
```

## 4. 查询优化技巧

### 4.1 查询重写优化

```sql
-- 原始查询（效率较低）
SELECT * FROM orders WHERE YEAR(create_time) = 2023;

-- 优化后的查询（使用索引）
SELECT * FROM orders
WHERE create_time >= '2023-01-01 00:00:00'
  AND create_time < '2024-01-01 00:00:00';
```

### 4.2 JOIN优化

```sql
-- 使用索引进行JOIN优化
SELECT o.order_no, c.customer_name
FROM orders o
JOIN customers c ON o.customer_id = c.id
WHERE o.status = 'completed';

-- 确保JOIN字段有索引
CREATE INDEX idx_customer_id ON orders(customer_id);
CREATE INDEX idx_id ON customers(id);
```

### 4.3 子查询优化

```sql
-- 原始子查询
SELECT * FROM orders
WHERE customer_id IN (SELECT id FROM customers WHERE level = 'VIP');

-- 优化为JOIN
SELECT o.*
FROM orders o
JOIN customers c ON o.customer_id = c.id
WHERE c.level = 'VIP';
```

## 5. 表结构优化

### 5.1 数据类型选择

```sql
-- 选择合适的数据类型
CREATE TABLE products (
    id INT UNSIGNED NOT NULL AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    price DECIMAL(10,2) NOT NULL,
    stock INT UNSIGNED DEFAULT 0,
    is_active TINYINT(1) DEFAULT 1,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    INDEX idx_name (name),
    INDEX idx_price (price)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;
```

### 5.2 字段规范

```cpp
// 字段选择原则
struct column_principle {
    bool use_unsigned;          // 使用无符号类型
    bool use_not_null;         // 使用NOT NULL约束
    bool use_default_value;    // 使用默认值
    bool avoid_varchar;        // 避免过长的VARCHAR
    bool use_enum_for_fixed;   // 固定值使用ENUM
};
```

## 6. 配置优化

### 6.1 内存配置

```sql
-- InnoDB缓冲池配置
SET GLOBAL innodb_buffer_pool_size = 4G;  -- 占用总内存的50-70%

-- 查询缓存（MySQL 8.0已移除）
-- SET GLOBAL query_cache_size = 256M;

-- 连接缓存
SET GLOBAL thread_cache_size = 16;
SET GLOBAL table_open_cache = 2000;
```

### 6.2 I/O配置

```sql
-- InnoDB I/O线程
SET GLOBAL innodb_read_io_threads = 8;
SET GLOBAL innodb_write_io_threads = 8;

-- 刷新策略
SET GLOBAL innodb_flush_log_at_trx_commit = 1;  -- 完全持久化
SET GLOBAL innodb_flush_method = 'O_DIRECT';  -- 直接I/O
```

### 6.3 连接配置

```sql
-- 最大连接数
SET GLOBAL max_connections = 1000;

-- 连接超时
SET GLOBAL wait_timeout = 28800;
SET GLOBAL interactive_timeout = 28800;

-- 最大允许数据包
SET GLOBAL max_allowed_packet = 256M;
```

## 7. 慢查询分析

### 7.1 慢查询配置

```sql
-- 开启慢查询日志
SET GLOBAL slow_query_log = ON;
SET GLOBAL slow_query_log_file = '/var/log/mysql/mysql-slow.log';
SET GLOBAL long_query_time = 2;  -- 超过2秒的查询
SET GLOBAL log_queries_not_using_indexes = ON;
```

### 7.2 慢查询分析工具

```sql
-- 使用mysqldumpslow分析
mysqldumpslow -s t /var/log/mysql/mysql-slow.log

-- 使用pt-query-digest分析
pt-query-digest /var/log/mysql/mysql-slow.log
```

### 7.3 常见慢查询模式

```sql
-- 1. 全表扫描
SELECT * FROM large_table WHERE status = 1;

-- 2. 索引失效
SELECT * FROM users WHERE phone_number = '13800138000';

-- 3. 排序操作
SELECT * FROM orders ORDER BY create_time DESC LIMIT 1000;

-- 4. 大批量插入
INSERT INTO large_table SELECT * FROM another_large_table;
```

## 8. 性能监控

### 8.1 关键指标监控

```sql
-- 查询性能指标
SHOW STATUS LIKE 'Com_select';
SHOW STATUS LIKE 'Slow_queries';

-- InnoDB性能指标
SHOW STATUS LIKE 'Innodb_buffer_pool_read%';
SHOW STATUS LIKE 'Innodb_row_lock%';

-- 连接状态
SHOW STATUS LIKE 'Threads_connected';
SHOW STATUS LIKE 'Max_used_connections';
```

### 8.2 Performance Schema

```sql
-- 启用Performance Schema
UPDATE performance_schema.setup_instruments
SET ENABLED = 'YES', TIMED = 'YES';

-- 查看SQL执行统计
SELECT * FROM performance_schema.events_statements_summary_by_digest
ORDER BY SUM_TIMER_WAIT DESC LIMIT 10;
```

### 8.3 sys schema

```sql
-- 查看最耗时的SQL
SELECT * FROM sys.statements_with_runtimes_in_95th_percentile;

-- 查看全表扫描的SQL
SELECT * FROM sys.statements_with_full_table_scans;

-- 查看索引使用情况
SELECT * FROM sys.schema_unused_indexes;
```

## 9. 实战优化案例

### 9.1 优化前：慢查询

```sql
-- 原始查询（执行时间：5.2秒）
SELECT o.order_no, c.customer_name, d.product_name, od.quantity
FROM orders o
JOIN customers c ON o.customer_id = c.id
JOIN order_details od ON o.id = od.order_id
JOIN products d ON od.product_id = d.id
WHERE o.create_time BETWEEN '2023-01-01' AND '2023-12-31'
  AND c.level = 'VIP'
ORDER BY o.create_time DESC
LIMIT 1000;
```

### 9.2 优化过程

```sql
-- 1. 分析执行计划
EXPLAIN SELECT ... -- 发现使用了全表扫描

-- 2. 添加缺失的索引
CREATE INDEX idx_customer_level ON customers(level);
CREATE INDEX idx_order_create_time ON orders(create_time);
CREATE INDEX idx_order_details_product_id ON order_details(product_id);

-- 3. 优化查询
SELECT o.order_no, c.customer_name, d.product_name, od.quantity
FROM orders o FORCE INDEX (idx_order_create_time)
JOIN customers c FORCE INDEX (idx_customer_level) ON o.customer_id = c.id
JOIN order_details od ON o.id = od.order_id
JOIN products d ON od.product_id = d.id
WHERE o.create_time >= '2023-01-01' AND o.create_time <= '2023-12-31'
  AND c.level = 'VIP'
ORDER BY o.create_time DESC
LIMIT 1000;
```

### 9.3 优化结果

```sql
-- 优化后（执行时间：0.08秒，提升了65倍）
Query_time: 0.080000  Lock_time: 0.000000
Rows_sent: 1000  Rows_examined: 1500
```

## 10. 总结

本篇文章深入探讨了：

1. **查询优化器**的工作原理和成本模型
2. **执行计划**的详细解读方法
3. **索引优化**的策略和最佳实践
4. **查询优化**的实用技巧
5. **表结构优化**的设计原则
6. **配置参数**的调优方法
7. **慢查询分析**的完整流程
8. **性能监控**的关键指标

## 11. 持续优化的建议

1. **定期监控**：建立性能监控体系
2. **慢查询分析**：每日分析慢查询日志
3. **索引维护**：定期重建索引
4. **容量规划**：预估数据增长趋势
5. **架构优化**：考虑读写分离、分库分表

## 12. 思考题

1. 如何判断一个查询是否需要优化？
2. 复合索引的最左前缀原则是什么？
3. 什么是覆盖索引？它有什么优势？
4. 如何分析SQL执行计划中的关键信息？

---

**作者提示**：性能优化是一个持续的过程，需要结合实际业务场景进行调优。建议在测试环境中验证优化效果后再应用到生产环境。