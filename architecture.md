# 产品架构
   数据仓库（UCloud Data Warehouse）是大规模并行处理数据仓库产品，基于开源的Greenplum开发的大规模并发、完全托管的PB级数据仓库服务。UDW Greenplum可以通过SQL让数据分析更简单、高效，为互联网、物联网、金融、电信等行业提供丰富的业务分析能力。支持MADlib扩展，客户可以在udw上使用MADlib的扩展功能，从而让机器学习变得简单，支持PostGIS，可以方便的支持空间、地理位置应用。最新支持greeplum6.2.1版本。


## 数据仓库产品架构

云数据库仓库 UDW Greenplum服务的架构图如下所示：

![image](/images/architecture1.png)

UDW Greenplum采用无共享的 MPP 架构，适用于海量数据的存储和计算。UDW 的架构如上图所示，主要有 Client、Master Node 和 Compute Node 组成。基本组成部分的功能如下：

1. Client：访问 UDW 的客户端
    * 支持通过 JDBC、ODBC、PHP、Python、命令行 Sql 等方式访问 UDW
2. Master Node：访问 UDW 数据仓库的入口
    * 接收客户端的连接请求
    * 负责权限认证
    * 处理 SQL 命令
    * 调度分发执行计划
    * 汇总 Segment 的执行结果并将结果返回给客户端
3. Compute Node：
    * Compute Node 管理节点的计算和存储资源
    * 每个 Compute Node 由多个 Segment 组成
    * Segment 负责业务数据的存储、用户 SQL 的执行

## 高可用

![image](/images/architecture2.png)

如上图所示：

1. Compute Node 中任一 Segment 都会有一个 Mirror Segment 备份到其他的 Compute Node 上，当 Primary Segment 出现不可用的时候会自动切换到 Mirror Segment，当 Primary Segment 恢复之后，Primary Segment 会自动恢复这期间的变更。
2. Master 节点是主从模式，当 Active Master 不可用时会自动切换到 Standby Master。
