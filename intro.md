

# 产品架构

## 云数据仓库产品架构

云数据库仓库UDW服务的架构图如下所示：

![image](/images/udw_jiagou.png)

UDW采用无共享的MPP架构，适用于海量数据的存储和计算。UDW的架构如上图所示，主要有Client、Master Node和Compute
Node组成。基本组成部分的功能如下

1.  Client:访问UDW的客户端、支持通过JDBC、ODBC、PHP、Python、命令行Sql访问UDW
2.  Master
    Node:访问UDW数据仓库的入口，它接收客户端的连接请求、负责权限认证、处理SQL命令、调度分发执行计划、汇总Fragment的执行结果并将结果返回给客户
3.  Compute Node：Compute Node管理节点的计算和存储资源，每个Compute
    Node有多个Fragment组成，Fragment负责业务数据的存储、用户SQL的执行

## 高可用

![image](/images/udw_reben.png)

如上图所示，Compute Node中的Fragment通过mirror备份到其他的Compute Node上。当primary
Fragment出现不可用的时候会自动切换到mirror Fragment、当primary Fragment恢复之后、primary
Fragment会自动恢复这期间的变更。除了Compute Node节点为Fragment配置镜像之外、UDW也会为Active
Node配置镜像，确保系统的变更信息不会丢失，提升系统的健壮性。当Active Master不可用时会自动切换到Standby
master。
