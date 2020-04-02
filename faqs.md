# FAQs

## 创建好数据仓库之后怎么连接到UDW？

请参考帮助文档“快速上手-\>连接数据仓库”部分介绍。<https://docs.ucloud.cn/udw/gettingstart>

## UDW支持从mysql导入数据吗？

支持。我们提供了一种从MySQL向UDW导入数据的工具。这个工具可以全量、也可以增量从MySQL导入到UDW单线程可以到达每秒4000-8000条,多线程可以达到4w-10w条每秒。具体使用请参考[mysql数据导入到udw.pdf](http://udw.cn-bj.ufileos.com/mysql%E6%95%B0%E6%8D%AE%E5%AF%BC%E5%85%A5%E5%88%B0udw.pdf)

## HDFS/Hive与UDW之间可以导入导出数据吗？

可以。 为了方便udw和hdfs/hive之间的数据导入和导出，我们提供个两种方案：

1.  用sqoop实现hdfs和udw直接的数据导入导出，使用方法请参考：[hdfs和hive中数据导入导出到udw](http://udw.cn-bj.ufileos.com/hdfs%E5%92%8Chive%E4%B8%AD%E6%95%B0%E6%8D%AE%E5%AF%BC%E5%85%A5%E5%AF%BC%E5%87%BA%E5%88%B0udw.pdf)
2.  创建hdfs外部表，使用方法请参考：[创建hdfs外部表](http://udw.cn-bj.ufileos.com/HDFS%E5%A4%96%E9%83%A8%E8%A1%A8.pdf)

## UDW中怎么kill掉正在执行的SQL语句？

查看哪些SQL语句正在执行，语句如下：

```
SELECT datname,procpid,query_start, current_query,waiting,client_addr FROM pg_stat_activity WHERE waiting='t';
```

> datname表示数据库名
> procpid表示当前的SQL对应的PID
> query_start表示SQL执行开始时间
> current_query表示当前执行的SQL语句
> waiting表示是否正在执行，t表示正在执行，f表示已经执行完成
> client_addr表示客户端IP地址

找出procpid，假设为1234。然后执行

```
SELECT pg_terminate_backend(1234);
```

## 如何通过外网访问UDW？

udw 默认是通过内网访问的，为了数据安全性，尽量不要通过外网访问 UDW，如果有访问外网的需求请参考：

前提：有一台可以访问 udw 的 uhost，并且这台 uhost 上可以访问外网 ip。

例如：uhost 内网 ip 是 10.10.0.9 外网 IP 是 192.168.120.110，udw 的访问ip 是
10.10.10.1， 我们在 uhost 机器上建立 ssh 隧道即可通过 192.168.120.110访问 udw。在 uhost
机器上执行如下命令：

```
ssh -C -f -N -g -L 5432:10.10.10.1:5432 root@10.10.0.9
```

备注：请注意开放外网防火墙端口 5432（也可以把 udw 端口映射到 uhost上其他端口上），网络防火墙配置请参考：<https://docs.ucloud.cn/unet/firewall/introduction>

## 节点扩容时数量有没有什么限制？

集群采用 Grouped Mirror 分布策略，因此新增节点数必须大于等于 2，以确保新增 Primary Segment 的 Mirror Segment 不在同一台机器上。

