



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
