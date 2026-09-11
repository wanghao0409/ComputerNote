  

# 1 Hadoop

**Hadoop** 是一个开源的<u>**分布式计算框架**</u>，专门用于存储和处理大规模数据集。
它最初由 Doug Cutting 和 Mike Cafarella 开发，灵感来源于 Google 的三篇经典论文（GFS、MapReduce 和 Bigtable）。
Hadoop 的核心设计目标是能够以低成本、高可靠性的方式处理海量数据。
    
# 2 **HDFS **
(Hadoop Distributed File System)

- **功能**
	- HDFS 是 Hadoop 的**分布式文件系统**，用于存储大规模数据。

- **优点**:

  - **海量数据存储：**典型文件大小GB、TB级别，百万以上文件数量
  - **高容错性**: 数据被<u>分割成块</u>（默认 128MB），并在集群中<u>**多节点复制**</u>（默认 **3** 份）。
  - **高吞吐量**: 适合大规模离线批量处理
  - **构建成本低、安全可靠**

- **缺点**：
  - 不适合低延迟数据访问
  - 不适合大量小文件存储（远远小于128M)
  - 不支持并发写入
  - 不支持文件随机修改、仅支持追加写入

# 3 主从架构


HDFS集群由**三个核心角色**组成：

  - **NameNode** (Master): 管理文件系统的<u>**元数据**</u>（如文件目录结构、块位置、文件属性）。

    - 备用节点
    - 心跳heartbeats,3秒检查一次dataNode的健康状态


  - **DataNode** (Slave): 存储实际数据块。
    block副本机制：
	- 数量：默认3份
      - 副本1：随机选择，优先空闲的DataNode节点
      - 副本2：放在不同的机架节点
      - 副本3：放在与第二个副本同一机架的不同节点
      - 副本N：随机选择
	- **目的**：防止机架断电或交换机故障，同时兼顾性能。
	
  - **SecondaryNameNode:**备用名称节点**。
	  - **并非热备！** 它定期合并NameNode的编辑日志（Edits）和镜像文件（FsImage），防止NameNode启动时加载过慢。注意：如果NameNode宕机，它并不能直接顶替（除非配置HA）。

> **高可用（HA）架构补充**：在生产环境中，现在通常配置 **Active/Standby NameNode**（使用ZKFC协调）。当Active宕机，Standby自动接管，实现秒级故障切换。


> 面试题：为什么dataNode为什么是128M？
>
> 答：块太大：寻址开销高，作业执行时间过长；
>
> ​		块太小：
>
> 

# 4 读写操作流程

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
> 1. **客户端**请求NameNode读取文件。
    
2. **NameNode**返回该文件所有块对应的DataNode地址列表（按离客户端远近排序）。
    
3. **客户端**直接连接最近的DataNode，读取块数据（并行读取不同块以提升速度）。
    

---
# 5 Shell常用操作（即学即用）

HDFS提供了类似Linux的Shell命令（通过 `hdfs dfs` 或 `hadoop fs` 执行）：

| 命令       | 作用             | 示例                                      |
| -------- | -------------- | --------------------------------------- |
| `-ls`    | 列出目录           | `hdfs dfs -ls /`                        |
| `-mkdir` | 创建文件夹          | `hdfs dfs -mkdir /user/data`            |
| `-put`   | 上传文件（从本地到HDFS） | `hdfs dfs -put ./local.txt /user/data/` |
| `-get`   | 下载文件（从HDFS到本地） | `hdfs dfs -get /user/data/hdfs.txt ./`  |
| `-cat`   | 查看文件内容         | `hdfs dfs -cat /file.txt`               |
| `-rm`    | 删除文件或目录        | `hdfs dfs -rm -r /tmp/old`              |
| `-du -h` | 查看目录大小         | `hdfs dfs -du -h /user`                 |
1. **可视化界面**：记住默认端口 `9870`（NameNode Web UI），在这里你能直观看到集群容量、DataNode状态和块信息。