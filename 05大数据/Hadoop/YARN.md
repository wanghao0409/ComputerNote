



# 前提：现代理论基础-谷歌三篇论文

**1、Google File System（GFS）--2003年***

  HDFS的设计灵感，解决了存储方案。

- 分布式文件系统

- 采用主从架构，主节点管理元数据，分节点储存数据块。

2、**MapReduce: Simplified Data Processing on Large Clusters - 2004年**

  MapReduce成为Hadoop的核心计算框架，简化了数据处理。

3、**Bigtable: A Distributed Storage System for Structured Data - 2006年**

- 结构化数据的高效存储和访问'
- HBase'

# 第一节、Hadoop

**Hadoop** 是一个开源的<u>**分布式计算框架**</u>，专门用于存储和处理大规模数据集。它最初由 Doug Cutting 和 Mike Cafarella 开发，灵感来源于 Google 的三篇经典论文（GFS、MapReduce 和 Bigtable）。Hadoop 的核心设计目标是能够以低成本、高可靠性的方式处理海量数据。

## 1. **HDFS (Hadoop Distributed File System)**

- **功能**: HDFS 是 Hadoop 的分布式文件系统，<u>**用于存储大规模数据**</u>。

- **优点**:

  - **海量数据存储：**典型文件大小GB、TB级别，百万以上文件数量
  - **高容错性**: 数据被<u>**分割成块**</u>（默认 128MB），并在集群中<u>**多节点复制**</u>（默认 **3** 份）。
  - **高吞吐量**: 适合大规模离线批量处理
  - **构建成本低、安全可靠**

- **缺点**：

  - 不适合低延迟数据访问
  - 不适合大量小文件存储（远远小于128M)
  - 不支持并发写入
  - 不支持文件随机修改、仅支持追加写入

- **特点：**主从架构**:**

  - **NameNode**: 管理文件系统的<u>**元数据**</u>（如文件目录结构、块位置、文件属性）。

    - 备用节点
    - 心跳heartbeats,3秒检查一次dataNode的健康状态

  - **DataNode**: 存储实际数据块。

    - block副本存放策略：
      - 副本1：随机选择，优先空闲的DataNode节点
      - 副本2：放在不同的机架节点
      - 副本3：放在与第二个副本同一机架的不同节点
      - 副本N：随机选择

    ![image-20250204144358438](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20250204144358438.png)



> 面试题：为什么dataNode为什么是128M？
>
> 答：块太大：寻址开销高，作业执行时间过长；
>
> ​		块太小：
>
> 



> 面试题：请简述一下HDFS读写操作流程？
>
> 答：一、写入文件操作流程：
>
> 1. **客户端发起写入请求**：
>    - 客户端调用 HDFS API（如 `FileSystem.create()`）发起文件写入请求。
> 2. **与 NameNode 通信**：
>    - 客户端向 NameNode 请求文件写入权限。
>    - NameNode 检查文件是否存在以及客户端是否有写入权限。
>    - 如果检查通过，NameNode 返回一组 DataNode 列表（用于存储数据块）。
> 3. **建立数据管道（Pipeline）**：
>    - 客户端根据 NameNode 返回的 DataNode 列表，建立一个数据写入管道。
>    - 管道通常由多个 DataNode 组成（默认 3 个副本）。
> 4. **数据分块写入**：
>    - 客户端将文件数据分成固定大小的块（默认 128MB 或 256MB）。
>    - 数据块通过管道依次写入各个 DataNode。
> 5. **确认写入成功**：
>    - 每个 DataNode 将数据写入本地磁盘，并向下一个 DataNode 发送数据。
>    - 最后一个 DataNode 确认写入成功后，依次向前传递确认信息。
>    - 客户端收到最终确认后，关闭文件写入流。
> 6. **更新元数据**：
>    - 客户端通知 NameNode 文件写入完成。
>    - NameNode 更新文件的元数据（如块的位置信息）。
>
> 二、读文件操作流程：
>
> 

## 2. **MapReduce**

- **功能**: MapReduce 是 Hadoop 的<u>**分布式计算框架**</u>，用于并行处理大规模数据集。
- **特点**:
  - **编程模型**: 将计算任务分为两个阶段：
    - **Map 阶段**: 对输入数据进行处理，生成<u>**键值对**</u>（key-value pairs）。
    - **Reduce 阶段**: 对 Map 的输出进行<u>**汇总和聚合**</u>。
  - **自动并行化**: 任务被分配到集群中的多个节点并行执行。
  - **容错性**: 如果某个节点失败，任务会自动重新分配到其他节点。

## 3. **YARN (Yet Another Resource Negotiator)**

- **功能**: YARN 是 Hadoop 的**资源管理**系统，负责集群资源的调度和管理。

- **特点**:
  - **解耦计算与资源管理**: YARN 将资源管理与任务调度分离，支持多种计算框架（如 MapReduce、Spark、Flink）。
  
  - **核心组件**:
    - **ResourceManager**: 全局资源管理器，负责分配集群资源。
    - **NodeManager**: 每个节点上的代理，负责管理单个节点的资源
    
    





# 
Hive知识点

# 一、概念理解：

- Hive 是建立在 Hadoop 之上的<u>**数据仓库工具，**</u>提供类似 SQL 的查询语言（HiveQL）。

- 主要功能是将 SQL 查询转换为 MapReduce 任务，在 Hadoop 上执行

- Hive 会自动将其转换为一系列 MapReduce 任务（或 Spark 任务），在 Hadoop 集群上运行。

  ```
  用户输入 SQL
         ↓
  Hive 将 SQL 转为 HiveQL
         ↓
  Hive 编译器将 HiveQL 转为 MapReduce/Tez/Spark 任务
         ↓
  任务提交到 Hadoop 集群执行
         ↓
  返回结果给用户
  ```

## Hive 将SQL转换为MapReduce过程

Hive是基于Hadoop的数据仓库工具，它将Hive SQL语句转化为MapReduce任务来执行，以下是具体的执行过程：

### 1. 词法和语法解析

- **词法分析**：Hive将输入的Hive SQL语句分解成一个个独立的词法单元（Token），例如关键字（SELECT、FROM、WHERE等）、表名、列名等。
- **语法分析**：根据词法分析得到的词法单元，使用预定义的语法规则构建抽象语法树（AST，Abstract Syntax Tree）。AST是一种以树状结构表示SQL语句语法结构的形式，它清晰地展示了语句中各个部分的层次关系和逻辑结构。

### 2. 语义分析

- **类型检查**：对AST中的表达式和操作进行类型检查，确保操作符和操作数的类型匹配。例如，在进行数值运算时，操作数必须是数值类型。
- **表和列的验证**：检查SQL语句中引用的表和列是否存在于Hive的元数据中。元数据存储了表的结构信息（如列名、数据类型等）、表的存储位置等。

### 3. 生成逻辑查询计划

- 基于语义分析后的AST，生成逻辑查询计划。逻辑查询计划是一种抽象的、与具体执行引擎无关的查询执行方案，它描述了查询的逻辑步骤，如过滤、投影、连接等操作的先后顺序。

### 4. 生成物理查询计划

- **选择执行引擎**：由于Hive支持多种执行引擎（如MapReduce、Tez、Spark等），在这一步会根据配置或用户指定选择MapReduce作为执行引擎。
- **将逻辑查询计划转化为物理查询计划**：将逻辑查询计划中的操作转化为MapReduce任务的具体步骤，包括确定Mapper和Reducer的任务分配、数据的分区和排序方式等。例如，将过滤操作放在Mapper阶段执行，以减少数据传输量；将连接操作放在Reducer阶段执行。

### 5. 任务调度和资源分配

- **任务调度**：Hive将生成的MapReduce任务提交给Hadoop的资源管理器（如YARN）进行调度。YARN负责确定任务的执行顺序和执行节点。
- **资源分配**：YARN根据任务的需求为每个MapReduce任务分配计算资源（如CPU、内存等）。

### 6. MapReduce任务执行

#### Mapper阶段

- **数据读取**：Mapper从HDFS中读取输入数据，数据通常以文件的形式存储。Hive会根据表的存储格式（如TextFile、SequenceFile等）将数据解析成键值对（Key-Value）的形式。
- **数据处理**：Mapper对读取的数据进行处理，执行过滤、投影等操作。例如，根据WHERE子句的条件过滤掉不符合条件的数据，选择需要的列。
- **数据输出**：Mapper将处理后的数据以键值对的形式输出，这些数据会被分区并发送到不同的Reducer。

#### Shuffle阶段

- **分区**：Mapper输出的数据根据键的哈希值被分区，每个分区的数据会被发送到不同的Reducer。
- **排序**：在每个分区内，数据会按照键的顺序进行排序，以便Reducer能够按顺序处理数据。
- **合并**：在数据传输过程中，会对相同键的数据进行合并，减少数据传输量。

#### Reducer阶段

- **数据接收**：Reducer接收来自不同Mapper的分区数据。
- **数据处理**：Reducer对接收的数据进行进一步处理，如聚合操作（SUM、COUNT等）、连接操作等。
- **数据输出**：Reducer将处理后的数据输出到HDFS中，作为查询结果。

### 7. 结果返回

- 当MapReduce任务执行完成后，Hive将查询结果返回给用户。结果可以是存储在HDFS中的文件，也可以直接在Hive客户端中显示。



### 面试题：Hive 和传统数据库的区别是什么？

- > **回答要点**：
  >
  > - **存储**：Hive 数据存储在 HDFS，传统数据库用本地存储。
  >
  > - **计算**：Hive 用 MapReduce/Spark，传统数据库有专用引擎。
  >
  > - **延迟**：Hive 适合批处理，传统数据库支持实时查询。





### 面试题：hive和Hadoop之间的关系？

> 答：“Hive 和 Hadoop 是紧密相关但定位不同的技术。Hadoop 是一个分布式计算框架，提供数据存储（HDFS）和分布式计算（MapReduce）能力，适合大规模数据的存储和批处理任务。而 Hive 是建立在 Hadoop 之上的数据仓库工具，提供类似 SQL 的查询语言（HiveQL），将 SQL 查询转换为 MapReduce 任务在 Hadoop 上执行。
>
> ​	Hive 的主要优势是<u>**降低开发门槛**</u>、提高开发效率和支持复杂查询，适合数据仓库、批处理分析和复杂查询场景。
>
> ​	然而，Hive 的查询性能依赖于底层的 MapReduce 引擎，延迟较高，不适合实时查询。因此，Hive 和 Hadoop 是互补的关系，Hive 依赖于 Hadoop 提供的基础设施，同时简化了 Hadoop 的使用。“



📌 **定位总结**：
 Hive 是一个 **离线批处理数据仓库系统**，适用于 T+1 报表、日志分析、数据挖掘等场景。

# 二 内部表和外部表

|              | **内部表（管理表）**                                         | **外部表**                                                   |
| ------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| **数据管理** | Hive管理内部表的数据，将数据存储在Hive的默认文件路径中       | 据由外部系统管理，Hive只通过元数据来引用数据。               |
| **删除操作** | 删除内部表时，Hive会删除表的元数据（存储在Metastore中）以及表中的数据（HDFS上的数据）。 | 删除外部表时，**只删除元数据**，不删除实际的数据。           |
| **使用场景** | 适用于临时数据或中间表，不需要长期保存的数据。               | 适用于多个系统共享数据的情况，比如由其他程序创建和处理的数据，希望即使删除Hive表也不会影响数据。 |
| **时间管理** | Hive 处理表和数据的生命周期                                  | 用户可以控制数据的生命周期，Hive 只是借用这些数据。          |



# 三 **分区和分桶**

- **分区（Partition）**：按目录划分数据（如 `dt=20231001`），加速按分区过滤的查询。
  - 分区（partitioning）通过列值将数据分割到不同目录，优化查询过滤（如分区裁剪）
  - 分区支持所有数据类型（如字符串、日期），并非仅限于数值列
  - `PARTITIONED BY` 子句创建分区表

- **分桶（Bucket）**：按<u>**哈希值**</u>将数据分散到固定数量的文件中，优化 JOIN 和采样效率

|              | 分区                                                         | 分桶                                                         |
| ------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 概念         | 分区是将表的数据按照指定的列（分区列）的值进行划分，在物理上表现为将数据存储在不同的目录下。例如，一个存储日志数据的表按照日期进行分区，不同日期的数据会被存放在不同的以日期命名的目录中。 | 分桶是将表的数据按照指定列的值进行哈希计算，根据哈希结果将数据分散到不同的文件（桶）中。例如，一个用户表按照用户 ID 进行分桶，用户 ID 经过哈希计算后会被分配到不同的桶文件里。 |
| 数据组织方式 | 区是基于分区列的值将数据划分成不同的逻辑部分，每个分区对应一个独立的目录，目录名通常包含分区列的值。例如，表 `sales` 按年份分区，会有类似 `year=2023`、`year=2024` 这样的目录。 | 分桶是在分区内部或者整个表的数据中，根据分桶列的哈希值将数据存储到不同的文件里。一个分区内可以有多个桶文件。例如，在 `year=2023` 这个分区下，可能会有 `bucket_00000`、`bucket_00001` 等多个桶文件。 |
| 粒度         | 分区的粒度相对较粗，通常用于处理大规模的数据划分，适合按照一些宏观的维度（如日期、地区等）来组织数据。 | 分桶的粒度更细，是在分区的基础上进一步细分数据，能更精确地存储和处理数据。 |
| 数据查询效率 | 在查询时，通过指定分区条件，可以快速定位到需要查询的分区目录，减少扫描的数据量。例如，查询 2023 年的销售数据，只需要扫描 `year=2023` 这个分区目录下的数据。 | 分桶可以提高数据的抽样效率和连接操作的性能。在进行表连接时，如果两个表都按照相同的列进行分桶，Hive 可以并行地对相同编号的桶进行连接操作，减少数据的移动和比较。 |
| 适用场景     | **按时间范围查询**：当需要按时间维度（如年、月、日）查询数据时，使用分区可以显著提高查询效率。例如，分析每天的网站访问日志，按日期进行分区，查询某一天的数据时只需要扫描对应日期分区下的数据。 <br />**按地域范围查询**：如果数据与地理位置相关，按地区进行分区可以方便地查询某个地区的数据。例如，分析不同城市的销售数据，按城市进行分区，查询某个城市的销售情况时只需要扫描该城市分区下的数据。 | **数据抽样**：在处理大规模数据时，需要对数据进行抽样分析。分桶可以方便地实现数据抽样，只需要选择部分桶进行分析即可。例如，从大量用户数据中抽取一部分用户进行用户行为分析。<br /> **表连接优化**：当进行两个大表的连接操作时，如果两个表都按照相同的列进行分桶，Hive 可以并行地对相同编号的桶进行连接操作，减少数据的移动和比较，提高连接性能。例如，用户表和订单表都按照用户 ID 进行分桶，在连接这两个表时可以提高效率。 |

```SQL
-- 创建分区表
CREATE TABLE sales (
    product_id INT,
    amount DOUBLE
)
PARTITIONED BY (year INT, month INT);

-- 插入数据到指定分区
INSERT OVERWRITE TABLE sales PARTITION (year = 2023, month = 1)
SELECT product_id, amount FROM temp_sales WHERE year = 2023 AND month = 1;

-- 创建分桶表
CREATE TABLE users (
    user_id INT,
    username STRING
)
CLUSTERED BY (user_id) INTO 10 BUCKETS;

-- 插入数据
INSERT OVERWRITE TABLE users SELECT user_id, username FROM temp_users;
```

 总结：**Hive 是“SQL on Hadoop”的开创者，Spark SQL 是它的高性能继承者**。





|          | 静态分区                                                  | 动态分区                                                    |
| -------- | --------------------------------------------------------- | ----------------------------------------------------------- |
| 概念     | 静态分区在插入数据时需要<u>**预先定义分区字段的值**</u>， | 而动态分区则根据数据中的某一列<u>**动态决定分区的值**</u>。 |
| 使用场景 | 适用于分区数较少且结构变化不大的场景                      | 适用于数据量大且分区数量变化较大的情况                      |
|          |                                                           |                                                             |



# 四、文件格式



|                  |                                | 特点                                                         | 优点             |
| ---------------- | ------------------------------ | ------------------------------------------------------------ | ---------------- |
| 文本文件格式     | TEXTFILE                       | 纯文本格式，可读性好 默认格式，不需要额外配置 支持压缩，但压缩比较低 性能较差，不支持块压缩 |                  |
|                  | CSV                            | 逗号分隔的文本文件 兼容Excel等工具 适合小数据量交换          |                  |
| 列式存储格式     | ORC（Optimized Row Columnar）  | 在ORC文件结构中，谓词下推（Predicate Pushdown）主要依赖于每个Stripe的尾部（Stripe Footer）中存储的各列最小值、最大值等统计信息。<br />这些统计信息允许ORC在查询时评估谓词条件（如范围查询），如果整个Stripe的数据都不满足条件，则直接跳过读取该Stripe，从而实现高效的数据过滤 |                  |
|                  | Parquet                        | **列式存储格式** 与Spark完美集成<br />  <br />高效的压缩编码 <br />跨语言支持 | 支持嵌套数据结构 |
| 行式存储格式     | SEQUENCEFILE                   |                                                              |                  |
|                  | AVRO                           |                                                              |                  |
| 行列混合存储格式 | RCFile（Record Columnar File） |                                                              |                  |



### 一、文本文件格式

#### 1. TEXTFILE（默认格式）

sql

```
-- 创建TEXTFILE格式表
CREATE TABLE text_table (
    id INT,
    name STRING,
    age INT
)
STORED AS TEXTFILE;

-- 或者显式指定
CREATE TABLE text_table (
    id INT,
    name STRING,
    age INT
)
ROW FORMAT DELIMITED
FIELDS TERMINATED BY ','
STORED AS TEXTFILE;
```



**特点：**

- 纯文本格式，可读性好
- 默认格式，不需要额外配置
- 支持压缩，但压缩比较低
- 性能较差，不支持块压缩

#### 2. CSV

sql

```
CREATE TABLE csv_table (
    id INT,
    name STRING,
    age INT
)
ROW FORMAT DELIMITED
FIELDS TERMINATED BY ','
STORED AS TEXTFILE;
```



**特点：**

- 逗号分隔的文本文件
- 兼容Excel等工具
- 适合小数据量交换

### 二、列式存储格式

#### 1. ORC（Optimized Row Columnar）



```sql
-- 创建ORC格式表
CREATE TABLE orc_table (
    id INT,
    name STRING,
    age INT
)
STORED AS ORC;

-- 带压缩的ORC表
CREATE TABLE orc_compressed_table (
    id INT,
    name STRING,
    age INT
)
STORED AS ORC
TBLPROPERTIES ("orc.compress"="SNAPPY");
```

在ORC文件结构中，谓词下推（Predicate Pushdown）主要依赖于每个Stripe的尾部（Stripe Footer）中存储的各列最小值、最大值等统计信息。这些统计信息允许ORC在查询时评估谓词条件（如范围查询），如果整个Stripe的数据都不满足条件，则直接跳过读取该Stripe，从而实现高效的数据过滤

Hive 中 ORC 文件格式相比 TextFile 格式的主要优势在于其列式存储设计，这能显著提升查询性能（通过谓词下推和只读取相关列）和压缩效率（由于列式数据的同质性）。

ORC在文件和Stripe级别存储了列的统计信息（如min/max），查询引擎可以利用这些信息跳过不包含目标数据的整个数据块，这个特性被称为谓词下推（Predicate Pushdown）。

**特点：**

- **高性能列式存储**
- 支持压缩（ZLIB, SNAPPY, LZO）
- 内置索引（轻量级索引）
- 支持谓词下推
- 支持ACID事务
- 支持复杂数据类型

**优势：**

- 查询性能优秀
- 存储效率高
- 适合分析型工作负载

#### 2. Parquet



```sql
-- 创建Parquet格式表
CREATE TABLE parquet_table (
    id INT,
    name STRING,
    age INT
)
STORED AS PARQUET;

-- 带压缩的Parquet表
CREATE TABLE parquet_compressed_table (
    id INT,
    name STRING,
    age INT
)
STORED AS PARQUET
TBLPROPERTIES ("parquet.compression"="SNAPPY");
```



**特点：**

- **列式存储格式**
- 与Spark完美集成
- 支持嵌套数据结构
- 高效的压缩编码
- 跨语言支持

**优势：**

- 在Spark生态中性能极佳
- 适合嵌套数据
- 生态系统完善

### 三、行式存储格式

#### 1. SEQUENCEFILE

sql

```
CREATE TABLE sequence_table (
    id INT,
    name STRING,
    age INT
)
STORED AS SEQUENCEFILE;
```



**特点：**

- Hadoop原生二进制格式
- 支持块压缩
- 可分割，适合MapReduce
- 性能优于TEXTFILE

#### 2. AVRO

sql

```
CREATE TABLE avro_table (
    id INT,
    name STRING,
    age INT
)
STORED AS AVRO;
```



**特点：**

- 支持Schema演化
- 二进制格式，压缩效率高
- 跨语言支持
- 适合数据序列化

### 四、RCFile（Record Columnar File）

sql

```
CREATE TABLE rc_table (
    id INT,
    name STRING,
    age INT
)
STORED AS RCFILE;
```



**特点：**

- 行列混合存储
- 先按行分块，块内按列存储
- Hive早期列式格式，逐渐被ORC取代







# 五、数据倾斜



|                                      |      |      |
| ------------------------------------ | ---- | ---- |
| 数据分布不均匀（**主键分布不均**：） |      |      |
| 关联操作（**大表与小表关联**：）     |      |      |
| 聚合操作                             |      |      |
| 数据采样                             |      |      |

