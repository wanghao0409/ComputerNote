## 一、执行计划基础概念

### 1. 什么是执行计划？

执行计划是数据库优化器生成的、<u>**描述**</u>如何执行SQL查询的"<u>**路线图**</u>"。它告诉我们：

- 数据如何被访问（全表扫描、索引扫描）
- 表如何连接（Nested Loop、Hash Join、Merge Join）
- 操作执行顺序
- 每个操作的代价估算

### 2. 执行计划的关键指标

- **Cost（代价）**：优化器估算的执行成本
- **Rows（行数）**：每个操作处理的行数估算
- **Time（时间）**：执行时间估算
- **Bytes（字节）**：数据量大小

## 二、数据库普通SQL优化

### 1. 获取执行计划

```sql
-- 普通执行计划
EXPLAIN SELECT * FROM users WHERE age > 25;

-- 详细执行计划（包含实际执行信息）
EXPLAIN ANALYZE SELECT * FROM users WHERE age > 25;
```



### 2. 执行计划关键操作符解读

| 操作符              | 含义         | 调优建议             |
| :------------------ | :----------- | :------------------- |
| **Seq Scan**        | 全表扫描     | 考虑添加索引         |
| **Index Scan**      | 索引扫描     | 通常良好             |
| **Index Only Scan** | 仅索引扫描   | 最优情况             |
| **Nested Loop**     | 嵌套循环连接 | 适合小表连接         |
| **Hash Join**       | 哈希连接     | 适合中等大小表       |
| **Merge Join**      | 合并连接     | 适合排序好的大数据集 |



## （一）SQL 语句方面的优化

### select 语句

   避免使用select * ，要尽量使用明确字段名

### join 连接

-   小表驱动大表，尽可能减少外层循环
-   on 关键字的连接条件确保有索引，
-   避免笛卡尔积

### where

-   筛选条件要尽可能是索引列
-   筛选条件最严格的写在前面，帮助优化器尽早过滤数据
-   使用合适的比较操作符 ，比如exists 比in的效果好

###  group by 

   确保分组列有索引

  避免在group by 中使用复杂的表达式

### union 

 **使用 `UNION ALL` 代替 `UNION`：** 如果不需要去重，`UNION ALL` 性能更高

### 子查询方面的优化

join比子查询效果好



## （二）数据层-方面的优化

### 1、**合理设计表结构**

- **遵循范式，但适度反范式：** 过度范式化会导致大量JOIN。在性能要求高的场景，可以适当冗余数据（反范式化）来减少JOIN。
- **选择合适的数据类型**：
  - 使用最小的合适类型（如用 `TINYINT` 代替 `INT` 存状态）。
  - 使用定长类型（如 `CHAR`）当长度固定时，效率更高。
- **避免使用 `TEXT` / `BLOB` 在频繁查询的表中：** 大字段会拖慢查询速度。考虑将大字段分离到单独的表中。
- 使用NOT Null 定义字段

### 2 **利用分区 (Partitioning)：**

- 对于超大表（如按时间、地域分区的日志表），分区可以显著提升查询性能（分区裁剪）和管理效率（如快速删除旧分区）。

  

## 三、HiveSQL优化

Hive 慢通常源于

- <u>**大规模数据的全表扫描**</u>
-  **<u>数据倾斜</u>**



### **一、数据层优化（治本之策）**

这是最有效、最根本的优化方式，主要影响 `Map` 阶段。

1. **分区**

   - **是什么**：根据某一列（通常是**日期`dt`**、**地区`region`**）的值将表数据存储在不同的目录中。

   - **为什么**：查询时通过 `WHERE` 条件指定分区，可以**避免全表扫描，只读取相关目录的数据**。

   - **怎么做**：

     ```
     -- 创建分区表
     CREATE TABLE logs (id INT, ...) PARTITIONED BY (dt STRING);
     -- 查询时务必使用分区过滤
     SELECT * FROM logs WHERE dt = '2023-10-25';
     ```

     

2. **分桶**

   - **是什么**：根据某一列的哈希值，将数据划分到固定数量的文件（桶）中。

   - **为什么**：

     - **高效采样**：可以快速对一小部分数据进行采样测试。
     - **提升 Join 效率**：如果两个表都以相同方式（相同列、相同桶数）分桶，在 Join 时可以转化为 **Sort-Merge-Bucket-Join**，极大减少 Shuffle 数据量。

     ```
     -- 创建分桶表
     CREATE TABLE user (id INT, ...) CLUSTERED BY (id) INTO 32 BUCKETS;
     -- 设置强制分桶属性
     SET hive.enforce.bucketing = true;
     ```

     

3. **使用合适的文件格式**

   - **列式存储**：`ORC` 和 `Parquet` 是首选。

     - **优点**：只读取查询涉及的列，IO 效率极高；自带索引和统计信息（如 min/max），支持谓词下推，在读取数据前就可以过滤掉不满足条件的块。

   - **压缩**：对文件进行压缩（如 Snappy, GZIP），减少磁盘存储和网络传输开销。

   - **怎么做**：

     sql

     ```
     CREATE TABLE ... STORED AS ORC tblproperties ("orc.compress"="SNAPPY");
     ```

4. **避免小文件**

   - **问题**：HDFS 不适合存储大量小文件，会导致 Map Task 数量过多，任务调度开销远超处理时间。
   - **解决**：
     - 在数据写入前，通过 `distribute by`、`sort by` 控制 reducer 数量，从而控制输出文件数。
     - 使用 `INSERT OVERWRITE ...` 语句重写表或分区，合并小文件。
     - 使用 `ALTER TABLE ... CONCATENATE` 命令（仅适用于 ORC 等格式）合并小文件。



### **二、查询层优化（SQL语句）**

1. **列裁剪**

   - **是什么**：只选择需要的列，避免 `SELECT *`。

   - **为什么**：特别是在列式存储中，能极大减少数据扫描量。

   - ```
     -- 不好
     SELECT * FROM orders;
     -- 好
     SELECT order_id, amount FROM orders;
     ```

     

2. **谓词下推**

   - **是什么**：尽早进行数据过滤，将 `WHERE` 条件在扫描数据时就应用。

   - **为什么**：Hive 的优化器会自动进行，但编写 SQL 时要为其创造条件。

   - **怎么做**：

     ```
     -- 过滤条件放在子查询中，尽早减少数据量
     SELECT a.*, b.name
     FROM (
         SELECT * FROM orders WHERE dt = '2023-10-25' AND status = 'paid'
     ) a
     JOIN users b ON a.user_id = b.id;
     ```

     

3. **优化 JOIN 操作**

   - **Map-Side Join**：

     - **是什么**：将小表完全加载到每个 Mapper 的内存中，在 Map 端完成 Join，避免 Shuffle。

     - **怎么做**：

       ```
       -- 设置自动选择 Map Join
       SET hive.auto.convert.join = true;
       -- 设置小表的大小阈值（默认~25MB）
       SET hive.mapjoin.smalltable.filesize = 25000000;
       -- 手动指定（Hive 0.7 前）
       SELECT /*+ MAPJOIN(b) */ a.key, a.value, b.name FROM a JOIN b ON a.key = b.key;
       ```

       

   - **处理 Join 数据倾斜**：

     - **问题**：当某个 Join Key 的值特别多时，会导致一个 Reducer 处理时间极长。

     - **解决**：

       ```
       -- 1. 开启倾斜 Join 优化
       SET hive.optimize.skewjoin = true;
       SET hive.skewjoin.key = 100000; -- 认为 key 的记录数超过 10w 就是倾斜
       -- 2. 将倾斜的 key 拆开处理（常用且高效）
       -- 假设 'key_A' 是倾斜的 key
       SELECT /*+ SKEW('table_a','key_A') */ ...
       FROM table_a a
       JOIN table_b b ON a.key = b.key;
       -- 或者手动拆分
       SELECT ... FROM a JOIN b ON a.key = b.key
       WHERE a.key != 'key_A'
       UNION ALL
       SELECT ... FROM a JOIN b ON a.key = b.key
       WHERE a.key = 'key_A';
       ```

       

------

### **三、计算与资源层优化（参数调优）**

1. **调整并行度**

   - **Map 数量**：通常由输入文件数和大小决定，可通过 `set mapred.max.split.size` 调整 split 大小来间接控制。

   - **Reduce 数量**：这是最常调整的参数之一。

     ```
     -- 直接设置 Reduce 任务个数
     SET mapreduce.job.reduces = 100;
     -- 或者根据数据量自动估算（推荐）
     SET hive.exec.reducers.bytes.per.reducer = 256000000; -- 每个 Reduce 处理 256MB
     ```

     

     - **原则**：Reduce 数量过多会导致小文件，过少会导致单个 Reduce 压力大。目标是让每个 Reduce 处理时间在几分钟到半小时内。

2. **启用并行执行**

   - **是什么**：Hive 会将一个查询解析成多个阶段（Stage），默认这些阶段是顺序执行的。开启并行后，非依赖的 Stage 可以同时执行。

   - **怎么做**：

     ```
     SET hive.exec.parallel = true;
     SET hive.exec.parallel.thread.number = 8; -- 设置并行度
     ```

     

3. **启用向量化查询**

   - **是什么**：一次处理一批数据（例如 1024 行），而不是一行一行处理，大幅提升 CPU 效率。

   - **前提**：数据格式必须是 ORC。

   - **怎么做**：

     ```
     SET hive.vectorized.execution.enabled = true;
     SET hive.vectorized.execution.reduce.enabled = true;
     ```

     

4. **启用 Cost-Based Optimizer**

   - **是什么**：Hive 的 CBO 会利用表和列的统计信息（如行数、NDV、数据分布）来生成更优的执行计划，例如选择最优的 Join 顺序。

   - **怎么做**：

     ```
     -- 收集统计信息
     ANALYZE TABLE table_name COMPUTE STATISTICS;
     ANALYZE TABLE table_name COMPUTE STATISTICS FOR COLUMNS column_name;
     -- 启用 CBO
     SET hive.cbo.enable = true;
     SET hive.compute.query.using.stats = true;
     SET hive.stats.fetch.column.stats = true;
     ```

## 四、SparkSQL优化

### **监控与诊断**

1. **Spark Web UI（最重要的工具）**

   - **Jobs & Stages**：查看任务整体执行时间和阶段划分。
   - **DAG Visualization**：可视化执行计划，重点关注 `Exchange`（Shuffle）和 `Sort` 等昂贵操作。
   - **SQL/DataFrame Tab**：直接定位到执行的 SQL，查看其物理计划和指标。
   - **Stage Details**：查看每个 Stage 下所有 Task 的耗时、GC 时间、Shuffle 读写量。**这是发现数据倾斜的关键**。
   - **Storage Tab**：查看缓存的数据量和内存使用情况。

2. **分析执行计划**

   - 使用 `df.explain(‘extended’)` 查看逻辑和物理计划。
   - 关注 `== Physical Plan ==` 部分，寻找 `Exchange`（shuffle）和 `Sort`。

3. **关键指标解读**

   - **Task 耗时分布**：如果大部分 Task 很快，但个别 Task 极慢，**极有可能是数据倾斜**。

   - **Shuffle Spill**：如果 `Shuffle Spill (Memory)` 和 `Shuffle Spill (Disk)` 数值很高，说明 Shuffle 数据量太大，内存不足，溢写到了磁盘，这会极大影响性能。

   - **GC Time**：如果 GC 时间很高，说明 JVM 内存压力大。

     

### **二、数据层优化**

1. **选择合适的数据源和格式**

   - **首选列式格式**：**Parquet** 或 **ORC**。它们支持列裁剪、谓词下推，压缩效率高。
   - **启用谓词下推**：这些格式默认会与 Spark 的 `spark.sql.parquet.filterPushdown` 等配置协同工作，在扫描时就过滤数据。

2. **分区与分桶**

   - **分区**：与 Hive 类似，根据常用过滤条件（如日期）进行分区，避免全表扫描。

   - **分桶**：Spark 也支持分桶，可以在 Presto/Trino 或 Spark 自身进行 Bucket Join 时提升性能。

     sql

     ```
     df.write.bucketBy(42, "user_id").sortBy("user_id").saveAsTable("bucketed_table")
     ```

     

### **三、SQL语句优化**

1. **避免 Shuffle（最高原则）**

   - **宽依赖操作**：`groupByKey`, `reduceByKey`, `join`, `distinct`, `repartition` 等都会引起 Shuffle。

   - **优化 Join**：

     - **广播 Join**：当一张小表足够小时，将其广播到所有 Executor 节点，避免大表的 Shuffle。这是最重要的 Join 优化。

       scala

       ```
       // 自动广播（推荐）
       spark.conf.set("spark.sql.autoBroadcastJoinThreshold", "10MB") // 默认 10MB
       
       // 手动指定
       val df1 = ... // 大表
       val df2 = ... // 小表
       import org.apache.spark.sql.functions.broadcast
       df1.join(broadcast(df2), Seq("key"))
       ```

       

2. **减少 Shuffle 数据量**

   - 在 `groupBy` 或 `join` 前，使用 `filter` 或 `select` 尽早过滤掉不需要的数据，减少需要 Shuffle 的数据量。
   - 在 Shuffle 操作前，使用 `repartition` 并指定一个合理的分区键，可能有助于避免后续的数据倾斜。

3. **使用高效的操作符**

   - `reduceByKey`/`aggregateByKey` 优于 `groupByKey`：因为前者会在 Map 端进行本地 Combine，减少 Shuffle 数据量。
   - `flatMap` 优于 `map` + `filter`。
   - 使用 `Window` 函数替代自连接等复杂操作。

### **四、配置参数调优（核心）**

#### **1. 资源与并行度配置**

- **`spark.executor.memory`**：Executor 的总内存。根据任务需求调整。
- **`spark.executor.cores`**：每个 Executor 的 CPU 核心数。通常设置为 3-5 个，以平衡并行度和 HDFS 吞吐量。
- **`spark.executor.instances`**：Executor 的个数。`(executor_instances * executor_cores)` 决定了集群的并发 Task 数。
- **`spark.sql.adaptive.coalescePartitions.enabled`**：**强烈建议设为 `true`**（AQE 的一部分），自动合并 Shuffle 后过小的分区。
- **`spark.sql.adaptive.enabled`**：**Spark 3.x 的核心特性，必须设为 `true`**。开启自适应查询执行。
- **`spark.sql.adaptive.skew.enabled`**：**必须设为 `true`**。开启自动处理数据倾斜。

**资源分配公式参考**：
`总并发Task数 = spark.executor.instances * spark.executor.cores`
这个数应该足够大以充分利用集群资源，但又不能太大导致调度开销过高。

#### **2. Shuffle 调优**

- **`spark.sql.shuffle.partitions`**：**最重要的参数之一**。控制 Shuffle 后（如 Join, Aggregate）的分区数。默认 200 通常不合适。
  - **设太小**：每个分区数据量过大，可能导致 OOM 且无法充分利用资源。
  - **设太大**：产生大量小 Task，调度开销大。
  - **建议**：开始时可以设为 `(executor_instances * executor_cores * 2-3)`。**开启 AQE 后，此参数可作为初始值，AQE 会动态调整**。
- **`spark.sql.adaptive.advisoryPartitionSizeInBytes`**：AQE 在优化 Shuffle 分区时，目标的分区大小。例如设为 64MB，AQE 会尽量让每个分区接近这个大小。

#### **3. 内存与执行优化**

- **`spark.memory.fraction`**：执行内存和存储内存占总堆内存的比例（默认 0.6）。如果 GC 频繁或频繁 Spill，可以适当调高（如 0.7）。
- **`spark.memory.storageFraction`**：`spark.memory.fraction` 中用于存储的比例（默认 0.5）。如果缓存的数据不多，可以适当调低，给执行内存更多空间。
- **`spark.serializer`**：始终使用 `org.apache.spark.serializer.KryoSerializer`。Kryo 序列化比 Java 序列化更快、更紧凑。
- **`spark.sql.codegen.wholeStage`**：**确保为 `true`**。全阶段代码生成（Tungsten 引擎的核心），将多个操作符编译成单个函数，大幅提升 CPU 效率。

### **五、处理特定问题**

#### **1. 数据倾斜**

**表现**：Stage 中 99% 的 Task 很快完成，但个别 Task 运行极慢。

**解决方案**：

- **方案A：使用 AQE 自动处理（首选）**

  - 确保 `spark.sql.adaptive.skew.enabled = true`。
  - AQE 会自动检测倾斜的分区，并将其拆分成更小的子分区进行处理。

- **方案B：加盐打散**

  - 对倾斜的 Key 进行随机打散。

  scala

  ```
  // 1. 给大表的倾斜key加随机前缀
  val skewedDF = largeDF
    .withColumn("salted_key", concat($"skewed_key", lit("_"), floor(rand() * 10)))
  
  // 2. 给小表扩容，生成对应的所有加盐key
  val saltRange = (0 until 10).toList
  val broadcastDF = spark.createDataFrame(saltRange).toDF("salt")
  val saltedSmallDF = smallDF
    .crossJoin(broadcastDF)
    .withColumn("salted_key", concat($"skewed_key", lit("_"), $"salt"))
  
  // 3. 用加盐后的key进行Join
  saltedLargeDF.join(saltedSmallDF, "salted_key")
  // 4. 最后对结果进行聚合，去除盐值
  ```

  

#### **2. GC 开销大**

**表现**：在 Spark UI 的 Stage 页面，Task 的 GC 时间很高。

**解决方案**：

- 使用 G1GC 垃圾回收器：`--conf "spark.executor.extraJavaOptions=-XX:+UseG1GC"`
- 增加 Executor 堆内存：`spark.executor.memory`
- 减少存储内存比例：`spark.memory.storageFraction`，给执行内存更多空间。
- 检查是否存在内存泄漏，避免在 RDD/DataFrame 操作中引用大对象。

## 五、跨引擎通用调优模式

### 1. 执行计划分析检查清单

**数据<u>扫描</u>阶段：**

- 是否使用了全表扫描？
- 分区过滤是否生效？
- 谓词下推是否工作？

**连接**操作：

- 连接算法是否最优？
- 小表是否被广播？
- 连接顺序是否合理？

**聚合操作：**

- 是否有Map端聚合？
- 数据倾斜是否处理？
- 内存设置是否充足？

**Shuffle操作：**

- 分区数是否合理？
- 数据分布是否均匀？
- 压缩是否启用？

### 2. 基于执行计划的性能诊断表

| 执行计划现象    | 可能问题        | 解决方案               |
| :-------------- | :-------------- | :--------------------- |
| **全表扫描**    | 缺少索引/分区   | 添加索引/分区          |
| **数据倾斜**    | 某些key数据过多 | Salting、AQE、调整分区 |
| **大表Shuffle** | 分区数不合理    | 调整shuffle分区数      |
| **内存溢出**    | 数据量太大      | 增加内存、分页处理     |
| **连接性能差**  | 连接算法不佳    | 广播小表、调整连接顺序 |

### 3. 监控和验证

**Spark UI监控：**

- 查看Stages执行时间
- 分析Shuffle Read/Write大小
- 检查Task执行时间分布

**Hive/数据库监控：**

- 查询执行时间
- 资源使用情况
- 慢查询日志分析