
# Beeline客户端

Beeline 是 Hive 官方推荐的基于 JDBC 的交互式命令行客户端，用于连接 **HiveServer2**。它替代了旧版废弃的 Hive CLI（基于 Thrift 的 `hive` 命令）

### 核心特点

- **纯 JDBC 访问**：通过 JDBC 驱动连接 HiveServer2，支持多种认证方式（Kerberos、LDAP、自定义等）。
    
- **多会话与并发**：HiveServer2 支持多客户端并发查询，而旧 CLI 需启动新进程且资源开销大。
    
- **高可用性**：可配合 ZooKeeper 实现 HiveServer2 的负载均衡与故障转移。
    
- **安全增强**：支持代理用户（impersonation），允许管理员控制权限。

```bash
# 直接连接 HiveServer2（默认端口 10000）
beeline -u "jdbc:hive2://<host>:10000/<database>" -n <username> -p <password>

# 使用 ZooKeeper 服务发现（CDH/HDP 常用）
beeline -u "jdbc:hive2://zk-ensemble:2181/;serviceDiscoveryMode=zooKeeper;zooKeeperNamespace=hiveserver2"

# 指定认证方式（如 LDAP）
beeline -u "jdbc:hive2://<host>:10000/default;auth=LDAP" -n <user> -p <pass>

# 连接后进入交互模式
beeline
!connect jdbc:hive2://<host>:10000 <username> <password>
```