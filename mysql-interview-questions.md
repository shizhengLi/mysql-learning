# MySQL 面试大全：从基础到专家级

## 目录
1. [基础概念面试题](#基础概念面试题)
2. [高级特性面试题](#高级特性面试题)
3. [性能优化面试题](#性能优化面试题)
4. [场景设计面试题](#场景设计面试题)
5. [源码级面试题](#源码级面试题)
6. [实战案例分析](#实战案例分析)

---

## 基础概念面试题

### 1. 什么是数据库？什么是关系型数据库？

**答案**：
- **数据库**：按照数据结构来组织、存储和管理数据的仓库
- **关系型数据库**：基于关系模型的数据库，数据以表格形式存储，表与表之间通过关系（外键）连接

### 2. MySQL有哪些存储引擎？各自的特点是什么？

**答案**：

| 存储引擎 | 特点 | 适用场景 |
|----------|------|----------|
| InnoDB | 支持事务、行级锁、外键、崩溃恢复 | 事务安全、高并发 |
| MyISAM | 读取速度快、不支持事务、表级锁 | 读密集型应用 |
| Memory | 数据存储在内存中、访问速度快 | 临时表、缓存 |
| Archive | 只支持插入和查询、压缩存储 | 日志、历史数据 |
| CSV | 以CSV格式存储 | 数据导入导出 |

### 3. 什么是SQL？SQL有哪些类型？

**答案**：
SQL（Structured Query Language）是结构化查询语言，分为：

- **DQL**：数据查询语言（SELECT）
- **DML**：数据操作语言（INSERT、UPDATE、DELETE）
- **DDL**：数据定义语言（CREATE、DROP、ALTER）
- **DCL**：数据控制语言（GRANT、REVOKE）
- **TCL**：事务控制语言（COMMIT、ROLLBACK）

### 4. 什么是主键？什么是外键？

**答案**：
- **主键**：表中唯一标识每条记录的字段，不能为空，必须唯一
- **外键**：用于建立两个表之间关系的字段，引用另一张表的主键

### 5. 什么是索引？索引有什么优缺点？

**答案**：
- **索引**：帮助数据库高效获取数据的数据结构，类似书籍的目录
- **优点**：大大提高查询速度，保证数据唯一性
- **缺点**：占用存储空间，降低写入性能，需要维护成本

### 6. 什么是事务？事务的ACID特性是什么？

**答案**：
- **事务**：作为单个逻辑工作单元执行的一系列操作
- **ACID特性**：
  - **原子性（Atomicity）**：事务要么全部完成，要么全部不完成
  - **一致性（Consistency）**：事务执行前后数据库状态一致
  - **隔离性（Isolation）**：事务之间互不干扰
  - **持久性（Durability）**：事务一旦提交，结果永久保存

### 7. 什么是范式？数据库有哪些范式？

**答案**：
**范式**：数据库设计的规范标准，主要范式包括：

- **第一范式（1NF）**：字段不可分
- **第二范式（2NF）**：满足1NF，非主键字段完全依赖主键
- **第三范式（3NF）**：满足2NF，非主键字段之间不存在传递依赖
- **BCNF**：更强的3NF

### 8. 什么是视图？视图有什么作用？

**答案**：
- **视图**：虚拟表，基于SQL查询结果集
- **作用**：简化复杂查询、数据安全性、逻辑数据独立性

### 9. 什么是存储过程？什么是触发器？

**答案**：
- **存储过程**：预编译的SQL语句集合，可重复调用
- **触发器**：在特定事件（INSERT、UPDATE、DELETE）发生时自动执行的程序

### 10. 什么是连接（JOIN）？有哪些类型？

**答案**：
- **内连接（INNER JOIN）**：返回两个表中匹配的记录
- **左连接（LEFT JOIN）**：返回左表所有记录，右表不匹配为NULL
- **右连接（RIGHT JOIN）**：返回右表所有记录，左表不匹配为NULL
- **全连接（FULL JOIN）**：返回两个表的所有记录

---

## 高级特性面试题

### 1. InnoDB和MyISAM有什么区别？

**答案**：

| 特性 | InnoDB | MyISAM |
|------|--------|--------|
| 事务支持 | 支持 | 不支持 |
| 外键约束 | 支持 | 不支持 |
| 锁机制 | 行级锁 | 表级锁 |
| 崩溃恢复 | 支持 | 不支持 |
| 表空间 | 共享表空间 | 独立表空间 |
| 索引结构 | 聚簇索引 | 非聚簇索引 |

### 2. 什么是MVCC？如何实现的？

**答案**：
**MVCC**（Multi-Version Concurrency Control）多版本并发控制：

- **实现原理**：通过保存数据的历史版本，实现读不阻塞写，写不阻塞读
- **版本链**：每行数据包含事务ID和回滚指针，形成版本链
- **Read View**：事务启动时的快照，决定可见哪些版本
- **删除标记**：删除操作实际是标记为删除，由purge线程清理

### 3. MySQL的隔离级别有哪些？各有什么特点？

**答案**：

| 隔离级别 | 脏读 | 不可重复读 | 幻读 | 特点 |
|----------|------|------------|------|------|
| READ UNCOMMITTED | ✓ | ✓ | ✓ | 最低级别，性能最高 |
| READ COMMITTED | ✗ | ✓ | ✓ | Oracle默认级别 |
| REPEATABLE READ | ✗ | ✗ | ✓ | MySQL默认级别 |
| SERIALIZABLE | ✗ | ✗ | ✗ | 最高级别，性能最低 |

### 4. 什么是死锁？如何避免和解决死锁？

**答案**：
- **死锁**：两个或多个事务互相持有对方需要的锁，导致无限等待
- **避免方法**：
  - 按固定顺序访问表
  - 保持事务简短
  - 使用合理的事务隔离级别
- **解决方法**：
  - 设置锁超时（innodb_lock_wait_timeout）
  - 死锁检测和自动回滚

### 5. 什么是B+树？为什么MySQL使用B+树？

**答案**：
**B+树特性**：
- 多路平衡查找树
- 所有数据都存储在叶子节点
- 叶子节点之间有指针连接
- 非叶子节点只存储键值

**MySQL选择B+树的原因**：
- 减少磁盘I/O（树的高度低）
- 适合范围查询（叶子节点有序）
- 查询效率稳定（O(log n)）
- 适合大数据量的索引

### 6. 什么是聚簇索引？什么是非聚簇索引？

**答案**：
- **聚簇索引**：叶子节点存储完整的行数据，每张表只有一个
- **非聚簇索引**：叶子节点存储主键值，需要回表查询完整数据

**区别**：
- 聚簇索引查询速度更快
- 聚簇索引占用空间更少
- 非聚簇索引可能产生回表操作

### 7. 什么是覆盖索引？有什么优势？

**答案**：
**覆盖索引**：查询的字段都被包含在索引中，不需要回表查询

**优势**：
- 减少磁盘I/O操作
- 提高查询性能
- 避免回表操作

### 8. 什么是索引下推（ICP）？

**答案**：
**索引下推**（Index Condition Pushdown）：
- 在存储引擎层面过滤数据，减少回表操作
- 只将符合WHERE条件的数据返回给Server层
- 显著提高查询性能

### 9. 什么是自适应哈希索引？

**答案**：
**自适应哈希索引**（Adaptive Hash Index）：
- InnoDB自动创建的内存索引结构
- 根据查询模式自动创建和调整
- 不需要手动维护
- 适用于等值查询

### 10. 什么是redo log？什么是undo log？

**答案**：
- **redo log**：重做日志，记录数据页的修改，用于崩溃恢复
- **undo log**：撤销日志，记录数据的历史版本，用于事务回滚和MVCC

**区别**：
- redo log是物理日志，undo log是逻辑日志
- redo log用于持久性，undo log用于原子性和一致性

---

## 性能优化面试题

### 1. 如何优化慢查询？

**答案**：
**优化步骤**：

1. **识别慢查询**
   ```sql
   -- 开启慢查询日志
   SET GLOBAL slow_query_log = ON;
   SET GLOBAL long_query_time = 2;
   ```

2. **分析执行计划**
   ```sql
   EXPLAIN SELECT * FROM table WHERE condition;
   ```

3. **优化策略**
   - 添加合适的索引
   - 优化查询语句
   - 重写子查询为JOIN
   - 使用覆盖索引

4. **监控效果**
   ```sql
   SHOW PROFILE FOR QUERY 1;
   ```

### 2. 如何选择合适的索引？

**答案**：
**索引选择原则**：

1. **选择唯一性高的字段**
   - 区分度越高，索引效果越好
   - 建议区分度 > 80%

2. **遵循最左前缀原则**
   - 复合索引中，查询条件包含左边字段
   - (A,B,C)索引支持A、A,B、A,B,C查询

3. **考虑查询频率**
   - 频繁查询的字段优先建索引
   - 避免过度索引

4. **考虑数据类型**
   - 整型索引比字符串索引效率高
   - 固定长度比可变长度效果好

### 3. 如何进行分页优化？

**答案**：
**传统分页问题**：
```sql
-- 大偏移量分页性能差
SELECT * FROM orders ORDER BY id LIMIT 100000, 10;
```

**优化方案**：

1. **基于ID的分页**
   ```sql
   SELECT * FROM orders WHERE id > last_id ORDER BY id LIMIT 10;
   ```

2. **使用JOIN优化**
   ```sql
   SELECT t.* FROM orders t
   JOIN (SELECT id FROM orders ORDER BY id LIMIT 100000, 10) tmp
   ON t.id = tmp.id;
   ```

3. **使用覆盖索引**
   ```sql
   SELECT * FROM orders
   WHERE id IN (SELECT id FROM orders ORDER BY id LIMIT 100000, 10);
   ```

### 4. 如何进行批量插入优化？

**答案**：
**优化策略**：

1. **批量插入替代单条插入**
   ```sql
   -- 差
   INSERT INTO table VALUES (1,'a');
   INSERT INTO table VALUES (2,'b');

   -- 好
   INSERT INTO table VALUES (1,'a'), (2,'b'), (3,'c');
   ```

2. **禁用索引和外键检查**
   ```sql
   ALTER TABLE table DISABLE KEYS;
   -- 执行批量插入
   ALTER TABLE table ENABLE KEYS;
   ```

3. **调整事务大小**
   ```sql
   -- 每批1000条
   START TRANSACTION;
   INSERT INTO table VALUES (...);
   -- 插入1000条后
   COMMIT;
   ```

### 5. 如何优化COUNT查询？

**答案**：
**COUNT优化策略**：

1. **使用COUNT(1)替代COUNT(*)**
   ```sql
   -- 推荐使用COUNT(1)
   SELECT COUNT(1) FROM table;
   ```

2. **使用近似统计**
   ```sql
   -- 使用近似值
   SELECT TABLE_ROWS FROM information_schema.TABLES
   WHERE TABLE_SCHEMA = 'database' AND TABLE_NAME = 'table';
   ```

3. **使用Redis缓存**
   ```sql
   -- 使用缓存存储计数
   SET key_count value
   ```

### 6. 如何进行内存优化？

**答案**：
**内存配置优化**：

```sql
-- InnoDB缓冲池大小（建议50-70%的内存）
SET GLOBAL innodb_buffer_pool_size = 4G;

-- 查询缓存（MySQL 8.0已移除）
-- SET GLOBAL query_cache_size = 256M;

-- 连接缓存
SET GLOBAL thread_cache_size = 16;
SET GLOBAL table_open_cache = 2000;

-- 排序缓冲区
SET GLOBAL sort_buffer_size = 2M;
SET GLOBAL join_buffer_size = 2M;
```

### 7. 如何进行磁盘I/O优化？

**答案**：
**I/O优化策略**：

1. **使用SSD硬盘**
   - 显著提高I/O性能
   - 减少磁盘寻道时间

2. **调整InnoDB参数**
   ```sql
   -- I/O线程数
   SET GLOBAL innodb_read_io_threads = 8;
   SET GLOBAL innodb_write_io_threads = 8;

   -- 刷新策略
   SET GLOBAL innodb_flush_log_at_trx_commit = 1;
   SET GLOBAL innodb_flush_method = 'O_DIRECT';
   ```

3. **文件系统优化**
   - 使用ext4或xfs文件系统
   - 调整文件系统参数

### 8. 如何进行连接优化？

**答案**：
**连接优化策略**：

```sql
-- 最大连接数
SET GLOBAL max_connections = 1000;

-- 连接超时
SET GLOBAL wait_timeout = 28800;
SET GLOBAL interactive_timeout = 28800;

-- 连接池配置
SET GLOBAL thread_cache_size = 16;
```

**连接池优化**：
- 使用连接池（如Druid、HikariCP）
- 设置合理的最大连接数
- 实现连接复用

### 9. 如何进行表结构优化？

**答案**：
**表结构优化策略**：

1. **选择合适的数据类型**
   ```sql
   -- 使用精确类型
   INT vs BIGINT
   DECIMAL(10,2) vs FLOAT
   ```

2. **字段规范**
   ```sql
   -- 使用NOT NULL约束
   CREATE TABLE users (
       id INT NOT NULL AUTO_INCREMENT PRIMARY KEY,
       name VARCHAR(50) NOT NULL
   );
   ```

3. **表分区**
   ```sql
   -- 按时间分区
   CREATE TABLE logs (
       id INT,
       log_time DATETIME,
       content TEXT
   ) PARTITION BY RANGE (YEAR(log_time)) (
       PARTITION p2023 VALUES LESS THAN (2024),
       PARTITION p2024 VALUES LESS THAN (2025)
   );
   ```

### 10. 如何进行监控和诊断？

**答案**：
**监控指标**：

```sql
-- 性能指标
SHOW STATUS LIKE 'Com_%';
SHOW STATUS LIKE 'Innodb_%';
SHOW STATUS LIKE 'Handler_%';

-- 慢查询分析
SHOW VARIABLES LIKE '%slow_query%';
SHOW PROCESSLIST;

-- Performance Schema
SELECT * FROM performance_schema.events_statements_summary_by_digest
ORDER BY SUM_TIMER_WAIT DESC LIMIT 10;

-- sys schema
SELECT * FROM sys.statements_with_runtimes_in_95th_percentile;
```

---

## 场景设计面试题

### 1. 设计一个电商系统的数据库架构

**答案**：
**核心表设计**：

```sql
-- 用户表
CREATE TABLE users (
    id BIGINT AUTO_INCREMENT PRIMARY KEY,
    username VARCHAR(50) UNIQUE NOT NULL,
    email VARCHAR(100) UNIQUE NOT NULL,
    password_hash VARCHAR(255) NOT NULL,
    phone VARCHAR(20),
    status TINYINT DEFAULT 1,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    INDEX idx_username (username),
    INDEX idx_email (email)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

-- 商品表
CREATE TABLE products (
    id BIGINT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(200) NOT NULL,
    description TEXT,
    price DECIMAL(10,2) NOT NULL,
    stock INT DEFAULT 0,
    category_id INT,
    status TINYINT DEFAULT 1,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    INDEX idx_name (name),
    INDEX idx_price (price),
    INDEX idx_category (category_id)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

-- 订单表
CREATE TABLE orders (
    id BIGINT AUTO_INCREMENT PRIMARY KEY,
    order_no VARCHAR(50) UNIQUE NOT NULL,
    user_id BIGINT NOT NULL,
    total_amount DECIMAL(12,2) NOT NULL,
    status VARCHAR(20) DEFAULT 'pending',
    payment_method VARCHAR(20),
    shipping_address TEXT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    FOREIGN KEY (user_id) REFERENCES users(id),
    INDEX idx_user_id (user_id),
    INDEX idx_order_no (order_no),
    INDEX idx_created_at (created_at)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

-- 订单详情表
CREATE TABLE order_items (
    id BIGINT AUTO_INCREMENT PRIMARY KEY,
    order_id BIGINT NOT NULL,
    product_id BIGINT NOT NULL,
    quantity INT NOT NULL,
    unit_price DECIMAL(10,2) NOT NULL,
    total_price DECIMAL(10,2) NOT NULL,
    FOREIGN KEY (order_id) REFERENCES orders(id),
    FOREIGN KEY (product_id) REFERENCES products(id),
    INDEX idx_order_id (order_id),
    INDEX idx_product_id (product_id)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;
```

**分库分表策略**：
- 按用户ID分库：`user_id % 16`
- 按订单时间分表：按月分表
- 读写分离：主库写入，从库读取

**缓存策略**：
- 热门商品缓存到Redis
- 用户会话信息缓存
- 订单状态缓存

### 2. 设计一个社交网络的消息系统

**答案**：
**消息表设计**：

```sql
-- 消息表
CREATE TABLE messages (
    id BIGINT AUTO_INCREMENT PRIMARY KEY,
    sender_id BIGINT NOT NULL,
    receiver_id BIGINT NOT NULL,
    content TEXT NOT NULL,
    message_type TINYINT DEFAULT 1, -- 1:文本, 2:图片, 3:视频
    status TINYINT DEFAULT 0, -- 0:未读, 1:已读
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (sender_id) REFERENCES users(id),
    FOREIGN KEY (receiver_id) REFERENCES users(id),
    INDEX idx_sender (sender_id),
    INDEX idx_receiver (receiver_id),
    INDEX idx_created_at (created_at)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

-- 消息会话表
CREATE TABLE conversations (
    id BIGINT AUTO_INCREMENT PRIMARY KEY,
    user1_id BIGINT NOT NULL,
    user2_id BIGINT NOT NULL,
    last_message_id BIGINT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    FOREIGN KEY (user1_id) REFERENCES users(id),
    FOREIGN KEY (user2_id) REFERENCES users(id),
    FOREIGN KEY (last_message_id) REFERENCES messages(id),
    UNIQUE KEY uk_users (user1_id, user2_id),
    INDEX idx_last_message (last_message_id)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;
```

**优化策略**：
- 按用户ID分库分表
- 使用Redis缓存未读消息数
- 消息异步推送
- 历史消息归档

### 3. 设计一个日志系统

**答案**：
**日志表设计**：

```sql
-- 主日志表（按天分表）
CREATE TABLE logs_202401 (
    id BIGINT AUTO_INCREMENT PRIMARY KEY,
    log_time DATETIME NOT NULL,
    level VARCHAR(20) NOT NULL, -- INFO, WARN, ERROR
    service_name VARCHAR(100) NOT NULL,
    trace_id VARCHAR(100),
    user_id BIGINT,
    message TEXT NOT NULL,
    stack_trace TEXT,
    INDEX idx_log_time (log_time),
    INDEX idx_level (level),
    INDEX idx_service (service_name),
    INDEX idx_trace_id (trace_id),
    INDEX idx_user_id (user_id)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4
PARTITION BY RANGE (TO_DAYS(log_time)) (
    PARTITION p20240101 VALUES LESS THAN (TO_DAYS('2024-01-02')),
    PARTITION p20240102 VALUES LESS THAN (TO_DAYS('2024-01-03'))
    -- 动态添加分区
);
```

**架构设计**：
- 使用ELK Stack进行日志处理
- 按服务、时间、级别多维度索引
- 日志压缩和归档策略
- 实时监控和告警

### 4. 设计一个高并发的计数器系统

**答案**：
**技术方案**：

1. **Redis计数器**
   ```java
   // 使用Redis的INCR命令
   redis.incr("counter:page_view:123");

   // 使用Lua脚本保证原子性
   String script = "local current = redis.call('GET', KEYS[1]); "
                  + "if current then "
                  + "   redis.call('SET', KEYS[1], current + ARGV[1]); "
                  + "else "
                  + "   redis.call('SET', KEYS[1], ARGV[1]); "
                  + "end "
                  + "return redis.call('GET', KEYS[1]);";
   ```

2. **MySQL计数器表**
   ```sql
   CREATE TABLE counters (
       id BIGINT AUTO_INCREMENT PRIMARY KEY,
       counter_key VARCHAR(100) UNIQUE NOT NULL,
       value BIGINT DEFAULT 0,
       updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
       INDEX idx_key (counter_key)
   ) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

   -- 使用UPDATE语句更新计数
   UPDATE counters SET value = value + 1 WHERE counter_key = 'page_view:123';
   ```

3. **分片计数器**
   ```sql
   -- 按分片存储
   CREATE TABLE counter_shards (
       id BIGINT AUTO_INCREMENT PRIMARY KEY,
       counter_key VARCHAR(100) NOT NULL,
       shard_id INT NOT NULL,
       value BIGINT DEFAULT 0,
       updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
       UNIQUE KEY uk_key_shard (counter_key, shard_id),
       INDEX idx_key (counter_key)
   ) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;
   ```

**优化策略**：
- 使用Redis缓存减少MySQL压力
- 分片处理避免单点性能瓶颈
- 定期同步数据到MySQL
- 实现降级和容错机制

### 5. 设计一个秒杀系统

**答案**：
**数据库设计**：

```sql
-- 商品库存表
CREATE TABLE seckill_products (
    id BIGINT AUTO_INCREMENT PRIMARY KEY,
    product_id BIGINT NOT NULL,
    stock_count INT NOT NULL,
    start_time DATETIME NOT NULL,
    end_time DATETIME NOT NULL,
    status TINYINT DEFAULT 1,
    INDEX idx_product_id (product_id),
    INDEX idx_time_range (start_time, end_time)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

-- 秒杀订单表
CREATE TABLE seckill_orders (
    id BIGINT AUTO_INCREMENT PRIMARY KEY,
    user_id BIGINT NOT NULL,
    product_id BIGINT NOT NULL,
    order_no VARCHAR(50) UNIQUE NOT NULL,
    status TINYINT DEFAULT 1,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (user_id) REFERENCES users(id),
    FOREIGN KEY (product_id) REFERENCES products(id),
    UNIQUE KEY uk_user_product (user_id, product_id),
    INDEX idx_user_id (user_id),
    INDEX idx_product_id (product_id),
    INDEX idx_created_at (created_at)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;
```

**技术方案**：

1. **Redis预减库存**
   ```java
   // Lua脚本保证原子性
   String script = "local stock = redis.call('GET', KEYS[1]); "
                  + "if tonumber(stock) <= 0 then "
                  + "   return 0; "
                  + "end; "
                  + "redis.call('DECR', KEYS[1]); "
                  + "return 1;";
   ```

2. **消息队列异步下单**
   ```java
   // 使用Kafka或RabbitMQ
   rabbitTemplate.convertAndSend("seckill.order.queue", orderMessage);
   ```

3. **数据库乐观锁**
   ```sql
   UPDATE seckill_products
   SET stock_count = stock_count - 1
   WHERE id = 1 AND stock_count > 0;
   ```

**防重和限流**：
- Redis分布式锁
- 接口限流（令牌桶、漏桶算法）
- 用户级别限购
- 前端验证码

---

## 源码级面试题

### 1. MySQL的启动流程是怎样的？

**答案**：
**主要启动步骤**：

```cpp
// sql/mysqld.cc
int main(int argc, char **argv) {
    // 1. 初始化全局变量
    my_init();

    // 2. 解析命令行参数
    load_defaults("my.cnf", groups, &argc, &argv);

    // 3. 初始化网络
    network_init();

    // 4. 初始化存储引擎
    ha_init();

    // 5. 初始化服务器组件
    init_server_components();

    // 6. 启动网络监听
    handle_connections_sockets();

    // 7. 主事件循环
    while (!shutdown_in_progress) {
        // 处理连接请求
        // 执行查询
        // 维护后台任务
    }
}
```

**关键组件初始化**：
- **网络层**：建立监听端口，准备接受客户端连接
- **存储引擎**：初始化InnoDB等存储引擎
- **缓冲池**：分配内存，初始化缓冲池
- **线程池**：创建工作线程，准备处理查询
- **日志系统**：初始化redo log、undo log等

### 2. MySQL如何处理SQL查询？

**答案**：
**查询处理流程**：

```cpp
// sql/sql_parse.cc
bool dispatch_command(THD *thd, enum enum_server_command command,
                     const char *packet, uint packet_length) {
    // 1. 解析SQL语句
    lex_start(thd);

    // 2. 词法分析
    if (yylex(&thd->lex, thd)) {
        // 词法分析失败
    }

    // 3. 语法分析
    if (MYSQLparse(thd)) {
        // 语法分析失败
    }

    // 4. 查询重写
    if (query_rewrite(thd)) {
        // 查询重写
    }

    // 5. 查询优化
    optimize_query(thd, thd->lex);

    // 6. 执行查询
    execute_query(thd, thd->lex);

    // 7. 返回结果
    send_result(thd);
}
```

**关键处理步骤**：
- **词法分析**：将SQL语句分解为token
- **语法分析**：构建语法树
- **语义分析**：检查表、字段是否存在
- **查询优化**：选择最优执行计划
- **执行计划**：生成执行操作序列
- **结果返回**：将结果返回给客户端

### 3. InnoDB的缓冲池是如何工作的？

**答案**：
**缓冲池架构**：

```cpp
// storage/innobase/buf/buf0buf.cc
struct buf_pool_t {
    ulint size;                // 缓冲池大小
    buf_chunk_t* chunks;       // 内存块
    buf_page_t* page_hash;     // 页哈希表
    buf_block_t* free_list;    // 空闲列表
    buf_block_t* flush_list;   // 刷新列表
    buf_block_t* LRU_list;     // LRU列表
};
```

**工作原理**：

1. **页读取**：
   - 检查页是否在缓冲池中
   - 如果不在，从磁盘读取
   - 加入LRU列表头部

2. **页替换**：
   - 使用LRU算法管理页面
   - 扫描LRU列表尾部
   - 淘汰不常用的页面

3. **页刷新**：
   - 脏页加入刷新列表
   - 后台线程定期刷新
   - 检查点时强制刷新

4. **并发控制**：
   - 使用读写锁保护页面
   - 支持多线程并发访问
   - 避免缓存不一致

### 4. InnoDB的锁机制是如何实现的？

**答案**：
**锁数据结构**：

```cpp
// storage/innobase/lock/lock0lock.cc
struct lock_t {
    trx_t* trx;                // 事务对象
    dict_index_t* index;       // 索引对象
    ulint type;                // 锁类型
    ulint mode;                // 锁模式
    lock_rec_t rec;            // 记录锁信息
    lock_t* hash;              // 哈希链
};
```

**锁实现机制**：

1. **表级锁**：
   - 意向锁（IS, IX）
   - 自增锁（AUTO-INC）
   - 使用哈希表管理

2. **行级锁**：
   - 记录锁（Record Lock）
   - 间隙锁（Gap Lock）
   - 临键锁（Next-Key Lock）

3. **锁冲突检测**：
   ```cpp
   // 检查锁兼容性
   bool lock_compatible(ulint mode1, ulint mode2) {
       if (mode1 == LOCK_NONE || mode2 == LOCK_NONE) {
           return true;
       }

       if (mode1 == LOCK_IS) {
           return mode2 == LOCK_IS || mode2 == LOCK_IX;
       }

       if (mode1 == LOCK_IX) {
           return mode2 == LOCK_IX;
       }

       if (mode1 == LOCK_S) {
           return mode2 == LOCK_S;
       }

       return false;
   }
   ```

4. **死锁检测**：
   - 构建等待图
   - 检测环路
   - 选择牺牲者

### 5. MySQL的复制原理是什么？

**答案**：
**复制架构**：

```cpp
// 复制流程
class MySQLReplication {
public:
    // 主库：记录binlog
    void log_transaction(const Transaction& trx) {
        // 1. 写入binlog cache
        binlog_cache.write(trx);

        // 2. 同步到磁盘
        if (sync_binlog == 1) {
            binlog_cache.sync();
        }

        // 3. 发送到从库
        send_to_slaves(binlog_cache);
    }

    // 从库：应用binlog
    void apply_binlog(const BinlogEvent& event) {
        // 1. 接收binlog
        receive_binlog(event);

        // 2. 应用事务
        apply_transaction(event);

        // 3. 更新position
        update_relay_log_info(event.position);
    }
};
```

**复制类型**：

1. **异步复制**：
   - 主库写binlog后立即返回
   - 从库异步拉取binlog
   - 可能丢失数据

2. **半同步复制**：
   - 主库等待至少一个从库确认
   - 减少数据丢失风险
   - 增加延迟

3. **同步复制**：
   - 所有从库都确认后才返回
   - 数据完全一致
   - 性能较差

**关键组件**：
- **binlog**：记录所有修改操作
- **relay log**：从库临时存储binlog
- **IO线程**：从主库拉取binlog
- **SQL线程**：应用binlog到从库

### 6. MySQL的查询优化器是如何工作的？

**答案**：
**优化器架构**：

```cpp
// sql/sql_optimizer.cc
class QueryOptimizer {
public:
    // 优化查询
    bool optimize(THD *thd, TABLE_LIST *tables) {
        // 1. 查询重写
        query_rewrite(thd);

        // 2. 生成逻辑计划
        logical_plan = generate_logical_plan(tables);

        // 3. 物理优化
        physical_plan = optimize_physical_plan(logical_plan);

        // 4. 成本估算
        cost = estimate_cost(physical_plan);

        // 5. 选择最优计划
        best_plan = select_best_plan(physical_plans);

        return best_plan;
    }

private:
    // 成本模型
    double estimate_cost(const JoinPlan& plan) {
        double cost = 0;

        // I/O成本
        cost += estimate_io_cost(plan);

        // CPU成本
        cost += estimate_cpu_cost(plan);

        // 内存成本
        cost += estimate_memory_cost(plan);

        return cost;
    }
};
```

**优化过程**：

1. **查询重写**：
   - 子查询转JOIN
   - 常量传播
   - 谓词下推

2. **访问路径选择**：
   - 选择索引
   - 决定连接顺序
   - 选择连接算法

3. **成本估算**：
   - 统计信息分析
   - 选择性估算
   - 成本模型计算

4. **计划生成**：
   - 动态规划算法
   - 贪心算法
   - 启发式规则

### 7. InnoDB的事务是如何实现的？

**答案**：
**事务管理器**：

```cpp
// storage/innobase/trx/trx0trx.cc
struct trx_t {
    trx_id_t id;              // 事务ID
    trx_state_t state;        // 事务状态
    ulint isolation_level;    // 隔离级别
    lock_t* locks;            // 持有的锁
    undo_no_t undo_no;        // Undo编号
    read_view_t* read_view;   // 读视图
};
```

**事务生命周期**：

1. **事务开始**：
   ```cpp
   trx_t* trx_start(THD *thd) {
       // 1. 分配事务ID
       trx->id = assign_trx_id();

       // 2. 设置隔离级别
       trx->isolation_level = thd->tx_isolation;

       // 3. 创建读视图
       trx->read_view = create_read_view(trx);

       // 4. 加入事务列表
       trx_list_add(trx);

       return trx;
   }
   ```

2. **事务执行**：
   ```cpp
   int trx_execute(THD *thd, trx_t* trx, const Query& query) {
       // 1. 记录undo log
       undo_log = write_undo_log(trx, old_data);

       // 2. 执行修改
       result = execute_query(query);

       // 3. 记录redo log
       redo_log = write_redo_log(trx, new_data);

       // 4. 获取必要的锁
       acquire_locks(trx, query);

       return result;
   }
   ```

3. **事务提交**：
   ```cpp
   int trx_commit(trx_t* trx) {
       // 1. 写入commit标记
       write_commit_log(trx);

       // 2. 刷新redo log
       flush_redo_log(trx);

       // 3. 释放锁
       release_locks(trx);

       // 4. 清理资源
       trx_cleanup(trx);

       return 0;
   }
   ```

4. **事务回滚**：
   ```cpp
   int trx_rollback(trx_t* trx) {
       // 1. 应用undo log
       apply_undo_log(trx);

       // 2. 释放锁
       release_locks(trx);

       // 3. 清理资源
       trx_cleanup(trx);

       return 0;
   }
   ```

### 8. MySQL的索引是如何实现的？

**答案**：
**B+树实现**：

```cpp
// storage/innobase/btr/btr0btr.cc
struct btr_cur_t {
    dict_index_t* index;       // 索引对象
    buf_block_t* block;        // 缓冲块
    rec_t* rec;                // 记录指针
    ulint page_no;             // 页号
    ulint offset;              // 页内偏移
};
```

**索引创建**：

```cpp
// 创建索引
int create_index(const IndexDef& def) {
    // 1. 分配索引空间
    index = allocate_index(def);

    // 2. 创建B+树结构
    create_btree(index);

    // 3. 构建索引数据
    build_index(index, table_data);

    // 4. 更新系统表
    update_index_metadata(index);

    return 0;
}
```

**索引查找**：

```cpp
// 索引查找
rec_t* index_lookup(dict_index_t* index, const dtuple_t* key) {
    // 1. 定位根页
    page_no = index->root_page_no;

    // 2. 遍历B+树
    while (page_no != FIL_NULL) {
        // 3. 读取页
        block = buf_page_get(page_no);

        // 4. 页内查找
        rec = page_search(block, key);

        // 5. 判断是否找到
        if (rec != NULL) {
            return rec;
        }

        // 6. 继续查找子页
        page_no = get_child_page_no(rec);
    }

    return NULL;
}
```

**索引维护**：

```cpp
// 插入索引
int index_insert(dict_index_t* index, rec_t* rec) {
    // 1. 查找插入位置
    cursor = index_lookup(index, rec);

    // 2. 检查页空间
    if (page_is_full(cursor->block)) {
        // 3. 页分裂
        split_page(cursor->block);
    }

    // 4. 插入记录
    page_insert(cursor->block, rec);

    // 5. 更新父节点
    update_parent(cursor->block);

    return 0;
}
```

### 9. MySQL的崩溃恢复机制是什么？

**答案**：
**恢复管理器**：

```cpp
// storage/innobase/log/log0recv.cc
class RecoveryManager {
public:
    // 崩溃恢复
    void crash_recovery() {
        // 1. 扫描redo log
        scan_redo_log();

        // 2. 应用redo log
        apply_redo_log();

        // 3. 回滚未完成事务
        rollback_incomplete_transactions();

        // 4. 重建缓冲池
        rebuild_buffer_pool();
    }

private:
    // 扫描redo log
    void scan_redo_log() {
        // 1. 查找检查点
        checkpoint = find_latest_checkpoint();

        // 2. 从检查点开始扫描
        start_lsn = checkpoint.lsn;

        // 3. 遍历redo日志
        while (has_more_log(start_lsn)) {
            log_event = read_log_event(start_lsn);

            // 4. 解析日志事件
            parse_log_event(log_event);

            start_lsn += log_event.length;
        }
    }

    // 应用redo log
    void apply_redo_log() {
        // 1. 按LSN顺序应用
        for (auto& event : redo_events) {
            // 2. 检查页是否需要恢复
            if (needs_recovery(event)) {
                // 3. 应用修改
                apply_page_modification(event);
            }
        }
    }

    // 回滚未完成事务
    void rollback_incomplete_transactions() {
        // 1. 识别未完成事务
        for (auto& trx : active_transactions) {
            if (trx->state != TRX_STATE_COMMITTED) {
                // 2. 应用undo log
                apply_undo_log(trx);

                // 3. 释放资源
                release_transaction_resources(trx);
            }
        }
    }
};
```

**恢复过程**：

1. **前滚（Roll Forward）**：
   - 从检查点开始应用redo log
   - 重做已提交的事务
   - 恢复数据页到最新状态

2. **回滚（Rollback）**：
   - 识别未完成的事务
   - 应用undo log
   - 回滚未提交的修改

3. **清理**：
   - 释放临时资源
   - 重建内存结构
   - 验证数据一致性

### 10. MySQL的统计信息是如何收集的？

**答案**：
**统计信息收集器**：

```cpp
// sql/sql_statistics.cc
class StatisticsCollector {
public:
    // 收集表统计信息
    void collect_table_stats(TABLE* table) {
        // 1. 统计记录数
        table->stats.records = count_records(table);

        // 2. 统计数据长度
        table->stats.data_length = calculate_data_length(table);

        // 3. 统计索引长度
        table->stats.index_length = calculate_index_length(table);

        // 4. 统计字段统计信息
        collect_column_stats(table);

        // 5. 更新系统表
        update_system_stats(table);
    }

    // 收集索引统计信息
    void collect_index_stats(TABLE* table, KEY* key) {
        // 1. 统计基数
        key->rec_per_key = estimate_cardinality(key);

        // 2. 统计空值数量
        key->nulls = count_null_values(key);

        // 3. 统计唯一值数量
        key->unique_values = count_unique_values(key);

        // 4. 采样分析
        analyze_index_distribution(key);
    }

private:
    // 估算基数
    ha_rows estimate_cardinality(KEY* key) {
        // 1. 全表扫描（小表）
        if (table->stats.records < 1000) {
            return count_distinct_values(key);
        }

        // 2. 采样估算（大表）
        sample_size = calculate_sample_size(table->stats.records);
        distinct_values = count_distinct_sample(key, sample_size);

        // 3. 外推估算
        estimated_values = (distinct_values * table->stats.records) / sample_size;

        return estimated_values;
    }
};
```

**统计信息类型**：

1. **表级统计**：
   - 记录数
   - 数据长度
   - 索引长度
   - 平均行长度

2. **索引统计**：
   - 基数（Cardinality）
   - 空值数量
   - 唯一值数量
   - 分布直方图

3. **字段统计**：
   - 最小值/最大值
   - 空值比例
   - 数据分布
   - 选择性

**收集方式**：

1. **自动收集**：
   ```sql
   -- 开启自动统计信息收集
   SET GLOBAL innodb_stats_persistent = ON;
   SET GLOBAL innodb_stats_auto_recalc = ON;
   ```

2. **手动收集**：
   ```sql
   -- 分析表
   ANALYZE TABLE table_name;

   -- 更新统计信息
   UPDATE TABLE table_name STATISTICS;
   ```

3. **采样收集**：
   - 大表使用采样技术
   - 减少收集成本
   - 保证估算精度

---

## 实战案例分析

### 案例1：慢查询优化实战

**问题场景**：
一个电商网站的订单查询页面响应很慢，经常超时。

**原始查询**：
```sql
SELECT o.order_no, c.customer_name, p.product_name, od.quantity
FROM orders o
JOIN customers c ON o.customer_id = c.id
JOIN order_details od ON o.id = od.order_id
JOIN products p ON od.product_id = p.id
WHERE o.create_time BETWEEN '2023-01-01' AND '2023-12-31'
  AND c.level = 'VIP'
  AND p.category_id = 5
ORDER BY o.create_time DESC
LIMIT 1000;
```

**执行计划分析**：
```sql
EXPLAIN SELECT ...
+----+-------------+-------+------+---------------+------+---------+------+-------+-------------+
| id | select_type | table | type | possible_keys | key  | key_len | ref  | rows  | Extra       |
+----+-------------+-------+------+---------------+------+---------+------+-------+-------------+
|  1 | SIMPLE      | o     | ALL  | NULL          | NULL | NULL    | NULL | 500000| Using where |
|  1 | SIMPLE      | c     | ALL  | PRIMARY       | NULL | NULL    | NULL | 10000 | Using where |
|  1 | SIMPLE      | od    | ALL  | order_id      | NULL | NULL    | NULL | 200000| Using where |
|  1 | SIMPLE      | p     | ALL  | PRIMARY       | NULL | NULL    | NULL | 50000 | Using where |
+----+-------------+-------+------+---------------+------+---------+------+-------+-------------+
```

**问题分析**：
1. 订单表全表扫描（type=ALL）
2. 客户表全表扫描
3. 订单详情表全表扫描
4. 产品表全表扫描
5. 没有使用任何索引

**优化方案**：

1. **添加缺失的索引**：
   ```sql
   -- 创建订单表索引
   CREATE INDEX idx_orders_create_time ON orders(create_time);
   CREATE INDEX idx_orders_customer_id ON orders(customer_id);

   -- 创建客户表索引
   CREATE INDEX idx_customers_level ON customers(level);

   -- 创建订单详情索引
   CREATE INDEX idx_order_details_product_id ON order_details(product_id);

   -- 创建产品表索引
   CREATE INDEX idx_products_category ON products(category_id);
   ```

2. **优化查询语句**：
   ```sql
   -- 使用覆盖索引
   SELECT o.order_no, c.customer_name, p.product_name, od.quantity
   FROM orders o FORCE INDEX (idx_orders_create_time)
   JOIN customers c FORCE INDEX (idx_customers_level) ON o.customer_id = c.id
   JOIN order_details od ON o.id = od.order_id
   JOIN products p ON od.product_id = p.id FORCE INDEX (idx_products_category)
   WHERE o.create_time >= '2023-01-01' AND o.create_time <= '2023-12-31'
     AND c.level = 'VIP'
     AND p.category_id = 5
   ORDER BY o.create_time DESC
   LIMIT 1000;
   ```

3. **分页优化**：
   ```sql
   -- 使用游标分页
   SELECT o.order_no, c.customer_name, p.product_name, od.quantity
   FROM orders o
   JOIN customers c ON o.customer_id = c.id
   JOIN order_details od ON o.id = od.order_id
   JOIN products p ON od.product_id = p.id
   WHERE o.create_time < ? AND o.create_time >= '2023-01-01'
     AND c.level = 'VIP'
     AND p.category_id = 5
   ORDER BY o.create_time DESC
   LIMIT 1000;
   ```

**优化结果**：
- **执行时间**：从12.5秒优化到0.08秒
- **扫描行数**：从760,000行减少到2,500行
- **资源消耗**：CPU使用率降低80%，I/O减少90%

### 案例2：高并发写入优化

**问题场景**：
一个日志收集系统，每秒需要处理10,000条日志写入，经常出现写入瓶颈。

**原始表结构**：
```sql
CREATE TABLE logs (
    id BIGINT AUTO_INCREMENT PRIMARY KEY,
    log_time DATETIME NOT NULL,
    level VARCHAR(20) NOT NULL,
    service_name VARCHAR(100) NOT NULL,
    message TEXT NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;
```

**问题分析**：
1. 单表写入压力大
2. 自增主键成为瓶颈
3. 索引维护开销大
4. 锁竞争严重

**优化方案**：

1. **分表分库**：
   ```sql
   -- 按服务分表
   CREATE TABLE logs_service_1 (
       id BIGINT AUTO_INCREMENT PRIMARY KEY,
       log_time DATETIME NOT NULL,
       level VARCHAR(20) NOT NULL,
       service_name VARCHAR(100) NOT NULL,
       message TEXT NOT NULL,
       created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
       INDEX idx_log_time (log_time),
       INDEX idx_level (level)
   ) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

   -- 按时间分表
   CREATE TABLE logs_202401 (
       id BIGINT AUTO_INCREMENT PRIMARY KEY,
       log_time DATETIME NOT NULL,
       level VARCHAR(20) NOT NULL,
       service_name VARCHAR(100) NOT NULL,
       message TEXT NOT NULL,
       created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
       INDEX idx_log_time (log_time),
       INDEX idx_level (level)
   ) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;
   ```

2. **批量写入优化**：
   ```java
   // 批量插入代码
   public void batchInsertLogs(List<Log> logs) {
       // 每1000条一批
       int batchSize = 1000;

       for (int i = 0; i < logs.size(); i += batchSize) {
           int end = Math.min(i + batchSize, logs.size());
           List<Log> batch = logs.subList(i, end);

           // 使用事务
           transaction.begin();
           try {
               // 批量插入
               String sql = "INSERT INTO logs (log_time, level, service_name, message) " +
                           "VALUES (?, ?, ?, ?)";

               jdbcTemplate.batchUpdate(sql, new BatchPreparedStatementSetter() {
                   @Override
                   public void setValues(PreparedStatement ps, int j) throws SQLException {
                       Log log = batch.get(j);
                       ps.setTimestamp(1, new Timestamp(log.getLogTime().getTime()));
                       ps.setString(2, log.getLevel());
                       ps.setString(3, log.getServiceName());
                       ps.setString(4, log.getMessage());
                   }

                   @Override
                   public int getBatchSize() {
                       return batch.size();
                   }
               });

               transaction.commit();
           } catch (Exception e) {
               transaction.rollback();
           }
       }
   }
   ```

3. **配置优化**：
   ```sql
   -- 增大InnoDB缓冲池
   SET GLOBAL innodb_buffer_pool_size = 8G;

   -- 增加redo log大小
   SET GLOBAL innodb_log_file_size = 1G;

   -- 优化刷新策略
   SET GLOBAL innodb_flush_log_at_trx_commit = 2;

   -- 增加并发连接数
   SET GLOBAL innodb_thread_concurrency = 0;
   ```

4. **异步处理**：
   ```java
   // 使用消息队列
   @KafkaListener(topics = "log.topic")
   public void processLog(String message) {
       // 解析日志
       Log log = parseLog(message);

       // 异步写入数据库
       CompletableFuture.runAsync(() -> {
           logService.insertLog(log);
       }, executor);
   }
   ```

**优化结果**：
- **写入性能**：从1,000条/秒提升到15,000条/秒
- **响应时间**：从200ms降低到10ms
- **系统稳定性**：避免了写入瓶颈和锁竞争

### 案例3：内存溢出问题排查

**问题场景**：
一个生产环境的MySQL实例频繁出现内存溢出，导致服务崩溃。

**症状**：
- MySQL进程内存使用率持续增长
- 经常出现OOM Killer杀死进程
- 查询响应时间逐渐变慢
- 系统频繁重启

**排查过程**：

1. **内存使用分析**：
   ```bash
   # 查看MySQL内存使用
   ps -aux | grep mysql

   # 查看内存分布
   cat /proc/$(pidof mysqld)/status

   # 查看内存映射
   pmap -x $(pidof mysqld)
   ```

2. **配置参数检查**：
   ```sql
   -- 查看内存相关配置
   SHOW VARIABLES LIKE '%buffer%';
   SHOW VARIABLES LIKE '%cache%';
   SHOW VARIABLES LIKE '%sort%';
   SHOW VARIABLES LIKE '%join%';
   ```

3. **Performance Schema分析**：
   ```sql
   -- 启用内存监控
   UPDATE performance_schema.setup_instruments
   SET ENABLED = 'YES', TIMED = 'YES'
   WHERE NAME LIKE 'memory/%';

   -- 查看内存使用
   SELECT * FROM performance_schema.memory_summary_global_by_event_name
   ORDER BY CURRENT_NUMBER_OF_BYTES_USED DESC LIMIT 10;
   ```

**问题发现**：
1. InnoDB缓冲池配置过大（16GB）
2. 连接数过多导致内存泄漏
3. 查询缓存未关闭
4. 排序缓冲区配置过大

**解决方案**：

1. **内存配置优化**：
   ```sql
   -- 调整缓冲池大小
   SET GLOBAL innodb_buffer_pool_size = 8G;

   -- 关闭查询缓存
   SET GLOBAL query_cache_type = 0;
   SET GLOBAL query_cache_size = 0;

   -- 优化线程缓存
   SET GLOBAL thread_cache_size = 32;

   -- 调整排序缓冲区
   SET GLOBAL sort_buffer_size = 1M;
   SET GLOBAL join_buffer_size = 1M;
   ```

2. **连接管理优化**：
   ```sql
   -- 限制最大连接数
   SET GLOBAL max_connections = 500;

   -- 设置连接超时
   SET GLOBAL wait_timeout = 300;
   SET GLOBAL interactive_timeout = 300;

   -- 使用连接池
   # application.properties
   spring.datasource.hikari.maximum-pool-size=20
   spring.datasource.hikari.minimum-idle=5
   spring.datasource.hikari.idle-timeout=300000
   ```

3. **查询优化**：
   ```sql
   -- 优化大查询
   -- 原查询
   SELECT * FROM large_table WHERE status = 1;

   -- 优化后
   SELECT id, name FROM large_table WHERE status = 1 LIMIT 1000;
   ```

4. **监控和告警**：
   ```bash
   # 监控脚本
   #!/bin/bash

   # 检查MySQL内存使用
   MEMORY_USAGE=$(ps -p $(pidof mysqld) -o %mem --no-headers)
   if [ $(echo "$MEMORY_USAGE > 80" | bc) -eq 1 ]; then
       echo "MySQL memory usage is high: $MEMORY_USAGE%"
       # 发送告警
   fi
   ```

**优化结果**：
- **内存使用**：从16GB降低到6GB
- **系统稳定性**：不再出现OOM Killer
- **响应时间**：查询响应时间稳定在50ms内
- **资源利用率**：CPU使用率正常，内存使用稳定

### 案例4：死锁问题解决

**问题场景**：
一个银行转账系统频繁出现死锁，导致转账失败。

**死锁日志**：
```sql
SHOW ENGINE INNODB STATUS;

------------------------
LATEST DETECTED DEADLOCK
------------------------
2024-01-15 10:30:25 0x7f8c8a7b7000
*** (1) TRANSACTION:
TRANSACTION 12345, ACTIVE 0 sec starting index read
mysql tables in use 1, locked 1
LOCK WAIT 3 lock struct(s), heap size 1136, 2 row lock(s)
MySQL thread id 100, OS thread handle 0x7f8c8a7b7000, query id 5000
UPDATE accounts SET balance = balance - 100 WHERE id = 1

*** (1) WAITING FOR THIS LOCK TO BE GRANTED:
RECORD LOCKS space id 0 page no 12345 n bits 72 index PRIMARY of table `bank`.`accounts`
trx id 12345 lock_mode X locks rec but not gap waiting
Record lock, heap no 2 PHYSICAL RECORD: n_fields 5; compact format; info bits 0
 0: len 4; hex 80000001; asc     ;;
 1: len 6; hex 00000000304d; asc     0M;;
 2: len 7; hex 80000001d30110; asc        ;;
 3: len 4; hex 800003e8; asc     ;;
 4: len 4; hex 8000012c; asc    ,;;

*** (2) TRANSACTION:
TRANSACTION 12346, ACTIVE 0 sec updating
mysql tables in use 1, locked 1
3 lock struct(s), heap size 1136, 2 row lock(s)
MySQL thread id 101, OS thread handle 0x7f8c8a7b8000, query id 5001
UPDATE accounts SET balance = balance + 100 WHERE id = 2

*** (2) HOLDS THE LOCK(S):
RECORD LOCKS space id 0 page no 12345 n bits 72 index PRIMARY of table `bank`.`accounts`
trx id 12346 lock_mode X locks rec but not gap
Record lock, heap no 2 PHYSICAL RECORD: n_fields 5; compact format; info info bits 0
 0: len 4; hex 80000001; asc     ;;
 1: len 6; hex 00000000304d; asc     0M;;
 2: len 7; hex 80000001d30110; asc        ;;
 3: len 4; hex 800003e8; asc     ;;
 4: len 4; hex 8000012c; asc    ,;;

*** (2) WAITING FOR THIS LOCK TO BE GRANTED:
RECORD LOCKS space id 0 page no 12345 n bits 72 index PRIMARY of table `bank`.`accounts`
trx id 12346 lock_mode X locks rec but not gap waiting
Record lock, heap no 3 PHYSICAL RECORD: n_fields 5; compact format; info bits 0
 0: len 4; hex 80000002; asc     ;;
 1: len 6; hex 00000000304e; asc     0N;;
 2: len 7; hex 80000001d30110; asc        ;;
 3: len 4; hex 800003e8; asc     ;;
 4: len 4; hex 8000012c; asc    ,;;

*** WE ROLL BACK TRANSACTION (1)
```

**问题分析**：
1. 两个事务以相反的顺序锁定账户
2. 事务1先锁定账户1，等待账户2
3. 事务2先锁定账户2，等待账户1
4. 形成死锁循环

**解决方案**：

1. **固定锁定顺序**：
   ```java
   // 转账方法
   public void transfer(long fromAccountId, long toAccountId, BigDecimal amount) {
       // 确保按固定顺序锁定账户
       long firstLockId = Math.min(fromAccountId, toAccountId);
       long secondLockId = Math.max(fromAccountId, toAccountId);

       try {
           // 锁定第一个账户
           Account firstAccount = accountRepository.lockById(firstLockId);

           // 锁定第二个账户
           Account secondAccount = accountRepository.lockById(secondLockId);

           // 执行转账逻辑
           if (fromAccountId == firstLockId) {
               firstAccount.setBalance(firstAccount.getBalance().subtract(amount));
               secondAccount.setBalance(secondAccount.getBalance().add(amount));
           } else {
               secondAccount.setBalance(secondAccount.getBalance().subtract(amount));
               firstAccount.setBalance(firstAccount.getBalance().add(amount));
           }

           // 保存更新
           accountRepository.save(firstAccount);
           accountRepository.save(secondAccount);

       } catch (Exception e) {
           // 处理异常
           throw new TransferException("转账失败", e);
       }
   }
   ```

2. **乐观锁实现**：
   ```java
   @Entity
   public class Account {
       @Id
       private Long id;

       private BigDecimal balance;

       @Version
       private Long version;
   }

   @Service
   public class TransferService {
       @Transactional
       public void transferWithOptimisticLock(Long fromId, Long toId, BigDecimal amount) {
           Account fromAccount = accountRepository.findById(fromId).orElseThrow();
           Account toAccount = accountRepository.findById(toId).orElseThrow();

           // 检查余额
           if (fromAccount.getBalance().compareTo(amount) < 0) {
               throw new InsufficientBalanceException();
           }

           // 更新余额
           fromAccount.setBalance(fromAccount.getBalance().subtract(amount));
           toAccount.setBalance(toAccount.getBalance().add(amount));

           // 保存更新
           try {
               accountRepository.save(fromAccount);
               accountRepository.save(toAccount);
           } catch (ObjectOptimisticLockingFailureException e) {
               // 乐观锁冲突，重试或抛出异常
               throw new ConcurrentModificationException("账户已被修改，请重试");
           }
       }
   }
   ```

3. **分布式锁**：
   ```java
   @Service
   public class TransferService {

       @Autowired
       private RedissonClient redissonClient;

       @Transactional
       public void transferWithDistributedLock(Long fromId, Long toId, BigDecimal amount) {
           // 获取分布式锁
           String lockKey = "transfer:lock:" + Math.min(fromId, toId);
           RLock lock = redissonClient.getLock(lockKey);

           try {
               // 尝试获取锁
               boolean locked = lock.tryLock(10, 30, TimeUnit.SECONDS);
               if (!locked) {
                   throw new TransferException("系统繁忙，请稍后重试");
               }

               // 执行转账逻辑
               performTransfer(fromId, toId, amount);

           } catch (InterruptedException e) {
               Thread.currentThread().interrupt();
               throw new TransferException("转账被中断");
           } finally {
               // 释放锁
               if (lock.isHeldByCurrentThread()) {
                   lock.unlock();
               }
           }
       }
   }
   ```

4. **死锁检测和重试**：
   ```java
   @Service
   public class TransferService {

       @Retryable(value = {DeadlockLoserDataAccessException.class}, maxAttempts = 3, backoff = @Backoff(delay = 100))
       @Transactional
       public void transferWithRetry(Long fromId, Long toId, BigDecimal amount) {
           try {
               performTransfer(fromId, toId, amount);
           } catch (CannotAcquireLockException | DeadlockLoserDataAccessException e) {
               // 记录日志
               log.warn("转账发生死锁，准备重试: from={}, to={}, amount={}", fromId, toId, amount);
               throw e;
           }
       }

       @Recover
       public void recoverTransfer(DeadlockLoserDataAccessException e, Long fromId, Long toId, BigDecimal amount) {
           log.error("转账重试失败: from={}, to={}, amount={}", fromId, toId, amount);
           throw new TransferException("转账失败，请稍后重试");
       }
   }
   ```

**优化结果**：
- **死锁频率**：从每天10次降低到几乎为零
- **转账成功率**：从95%提升到99.9%
- **系统稳定性**：避免了死锁导致的服务中断
- **用户体验**：转账操作更加可靠

---

## 总结

这个MySQL面试题大全涵盖了从基础概念到源码级深度的全面内容，包括：

### 📚 知识体系
1. **基础概念**：数据库原理、SQL语法、索引基础
2. **高级特性**：InnoDB存储引擎、事务机制、MVCC
3. **性能优化**：查询优化、索引策略、配置调优
4. **场景设计**：电商系统、日志系统、高并发场景
5. **源码分析**：启动流程、查询处理、锁机制
6. **实战案例**：慢查询优化、高并发写入、内存问题、死锁处理

### 💡 学习建议
1. **循序渐进**：从基础到高级，逐步深入
2. **实践结合**：理论学习与实际操作相结合
3. **源码阅读**：通过源码理解底层实现
4. **案例分析**：通过真实案例掌握问题解决能力
5. **持续学习**：关注MySQL最新发展和技术趋势

### 🎯 准备策略
1. **基础扎实**：掌握核心概念和原理
2. **深入理解**：不仅知道是什么，还要知道为什么
3. **实践经验**：具备实际问题的解决能力
4. **表达能力**：能够清晰表达技术方案
5. **思维扩展**：具备系统设计和架构能力

希望这个面试题大全能够帮助你全面掌握MySQL知识，在面试中取得好成绩！