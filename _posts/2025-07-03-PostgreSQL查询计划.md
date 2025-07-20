PostgreSQL查询执行


在 PostgreSQL 中，查看 SQL 语句的执行计划主要使用 `EXPLAIN` **命令**。以下是详细方法和常用技巧：

### 1. **基础用法**

```sql
EXPLAIN SELECT * FROM your_table WHERE condition;
```
  显示查询的**预估执行计划**（不会实际执行查询）。
  输出包括执行节点类型（如顺序扫描、索引扫描）、预估行数和成本（cost）。

### 2. **实际执行分析（推荐）**

添加 `ANALYZE` 选项会**实际执行查询**并返回详细统计信息：

```sql
EXPLAIN ANALYZE SELECT * FROM your_table WHERE condition;
```
  输出包括：
    实际执行时间（毫秒）。
    实际返回行数。
    内存/缓存使用情况（如 `Buffers`）。

  ⚠️ 注意：此操作会实际执行查询，对写操作（INSERT/UPDATE/DELETE）需谨慎！

### 3. **常用高级选项**

通过参数扩展功能（逗号分隔）：

```sql
EXPLAIN (选项) SELECT ...;
  `**VERBOSE**`：显示额外信息（如输出列名）。
  EXPLAIN (VERBOSE) SELECT ...;
  `**BUFFERS**`：需配合 `ANALYZE`，显示缓存命中率（`shared hit` 表示内存读取）。
  EXPLAIN (ANALYZE, BUFFERS) SELECT ...;
  `**FORMAT**`：指定输出格式（支持 TEXT/JSON/XML/YAML）。
  EXPLAIN (ANALYZE, FORMAT JSON) SELECT ...;
```

### 4. **关键执行计划解读**

  **常见节点类型**：

    `Seq Scan`：全表顺序扫描（通常需优化）。    
    `Index Scan`：索引扫描（高效）。    
    `Bitmap Index Scan`：位图索引扫描（多条件组合）。    
    `Nested Loop`/`Hash Join`/`Merge Join`：表连接方式。

  **关键指标**：

    `cost`：预估成本（启动成本..总成本）。
    `rows`：预估返回行数。
    `actual time`：实际执行时间（`ANALYZE` 输出）。
    `Buffers: shared hit`：内存缓存命中率（越高越好）。

### 5. **优化示例**

```sql
-- 未使用索引（Seq Scan）
EXPLAIN ANALYZE SELECT * FROM users WHERE age > 30;

-- 创建索引后（Index Scan）
CREATE INDEX idx_users_age ON users(age);
EXPLAIN ANALYZE SELECT * FROM users WHERE age > 30;
```

### 6. **其他技巧**

  **禁止索引**（测试全表扫描代价）：
  ```sql
  SET enable_indexscan = OFF;
  EXPLAIN SELECT ...;
  ```
  **保存分析结果**：

  ```sql
  EXPLAIN (ANALYZE, FORMAT JSON) 
  INTO outfile '/path/output.json'
  SELECT ...;
  ```

### 执行计划优化重点：
1. 1. 警惕高 `cost` 或 `actual time` 的节点。
2. 2. 检查是否缺少索引（避免全表扫描 `Seq Scan`）。
3. 3. 关注 `rows` 预估是否准确（统计信息过时可运行 `ANALYZE table_name;` 更新）。

通过分析执行计划，可快速定位性能瓶颈并针对性优化！