# 产品架构

## 云数据仓库产品架构

云数据库仓库 UDW 服务的架构图如下所示：

![image](/images/architecture1.png)

UDW 采用无共享的 MPP 架构，适用于海量数据的存储和计算。UDW 的架构如上图所示，主要有 Client、Master Node 和 Compute Node组成。基本组成部分的功能如下：

1. Client：访问 UDW 的客户端
    * 支持通过 JDBC、ODBC、PHP、Python、命令行 Sql 等方式访问 UDW
2. Master Node：访问 UDW 数据仓库的入口:
    * 它接收客户端的连接请求
    * 负责权限认证
    * 处理 SQL 命令
    * 调度分发执行计划
    * 汇总 Segment 的执行结果并将结果返回给客户端
3. Compute Node：
    * Compute Node 管理节点的计算和存储资源
    * 每个 Compute Node 有多个 Segment 组成
    * Segment 负责业务数据的存储、用户 SQL 的执行

## 高可用

![image](/images/architecture2.png)

如上图所示：

1. Compute Node 中任意 Segment 会有一个 Mirror Segment 备份到其他的 Compute Node 上，当 Primary Segment 出现不可用的时候会自动切换到 Mirror Segment，当 Primary Segment 恢复之后，Primary Segment 会自动恢复这期间的变更。
2. 除了 Compute Node 节点为 Segment 配置镜像之外、UDW也会为 Master Node 配置镜像，确保系统的变更信息不会丢失，提升系统的健壮性，当 Active Master 不可用时会自动切换到 Standby Master。
