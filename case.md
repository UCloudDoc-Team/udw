{{indexmenu_n>800}}

# 数据分析案例

## 案例一 利用 logstash+Kafka+UDW 对日志数据分析

Logstash 是目前比流行、使用较多的日志收集和管理系统，Kafka也是企业常用的分布式发布-订阅消息系统，UDW（UCloud Data
Warehouse）是大规模并行处理数据仓库产品，下面介绍一些利用 logstash+Kafka+UDW 构建日志收集-存储-分析的全套解决方案。

### Logstash收集日志到Kafka

1. 下载安装https://www.elastic.co/downloads/logstash

2. logstash依赖java环境、确保已经安装过java

3. 安装logstash-output-kafka插件

4. 配置logstash收集日志写入Kafka

参考配置如下（更多参数和含义请参考官方文档）：

![image](/images/case1.1.png)

5. 启动logstash收集日志到Kafka

执行 `bin/logstash agent -f logstash-output-kafka.conf` 发送消息到 Kafka

备注：我们除了用 logstash 收集日志到 kafka 之外，我们还可以使用 Flume 收集日志到 Kafka，也可以把 Spark、Storm 中的流式数据写入到 Kafka。更多 kafka 的使用请参考：

<https://static.ucloud.cn/6799401b027e12e2206591051a107507.pdf>

### Kafka数据写入到UDW

下面以Nodejs语言为例、其他语言也可以实现消费Kafka的数据并且写入到UDW。

![image](/images/case1.2.png)

如上所示，我们可以定时的把Kafka消费的数据导入到UDW，上面的示例是用nodejs实现的，需要依赖kafka-node，kafka-node的git地址：<https://github.com/SOHU-Co/kafka-node>

下面是通过copy的方式把从Kafka中消费的数据导入到UDW，为了加快数据插入UDW的速度，强烈建议用copy的方式导入数据到UDW。nodejs的copy数据到postgresql的使用方法请参考：<https://github.com/brianc/node-pg-copy-streams>。

![image](/images/case1.3.png)

UDW的数据库和表格相关设计和操作，请参考：[UDW开发指南](https://docs.ucloud.cn/analysis/udw/developer)

### 利用UDW进行数据分析

#### 设计表格

在数据分析场景下、往往数据入库之后很少更新，我们创建一个列存储+压缩+分区表的表格。列存储、压缩、分区可以减少磁盘IO从而减少查询和分析时处理的数据量，大大提高查询效率。下面是orders表的建表实例。另外根据查询条件可以适当的建立索引从而进一步优化查询效率。

![image](/images/case1.4.png)

#### 数据分析

UDW提供了标准的Postgresql的SQL，如下所示，我们可以通过SQL就很方便的实现对数据的分析。

![image](/images/case1.5.png)

![image](/images/case1.6.png)

### 数据可视化

为了方便UDW的查询数据可视化话，我们可以把UDW接入第三方的BI系统，请参考我们的文档：
[UDW接入第三方BI系统](https://docs.ucloud.cn/analysis/udw/%E8%BF%9E%E6%8E%A5bi%E7%B3%BB%E7%BB%9F)

## 案例二 基于UDW实现网络流分析

### 背景介绍

网络流分析主要包括对用户的网络流数据进行存储和多维度的分析两部分。用户的网络流的数据每天产生400G左右，数据保留10天。针对网络流数据的分析主要包含流量分析、包量分析、TCP延迟分析、HTTP状态码分析、TCP重传分析等。

### 数据存储

创建了一个10个节点的UDW集群，节点类型为ds1.large,磁盘类型SATA，大小为2000GB。（由于UDW集群的高可用方案，集群可用大小为10000GB）。

### 表格设计

采取列存储和压缩的追加表，分布键为id,根据time分区,时间间隔为1天。完整的建表语句如下所示：

```
create table t_unetanalysis_data (
    id serial,uuid varchar(64) DEFAULT NULL,
    item_id int DEFAULT NULL,
    time int DEFAULT NULL,
    data varchar(4000) DEFAULT NULL)
with (APPENDONLY=true, ORIENTATION=column, compresslevel=5) DISTRIBUTED BY (id)
partition by range(time) (
    START (1469980800) END (1488297600) EVERY (86400)
);
```

其中，id 为记录序号，通过 serial（序列）实现自增；uuid 存储用户组织 ID 或者用户的 IP；item\_id 为代表某种分析项的 id（分析项如IP流量、TCP包量、TCP重传率等）；time为时间戳；data为数据。

样本数据如下图所示：

![image](/images/case2.1.png)

根据查询需要创建索引如下：

```
create index t_unetanalysis_data_uuid_itemid_time on t_unetanalysis_data(uuid, item_id, time);
```

### 数据导入

为了加快数据插入UDW的速度，强烈建议用copy的方式将数据导入。示例代码如下图所示：

![image](/images/case2.2.png)

执行 `python test_copy_to_udw.py`，输出如下：

![image](/images/case2.3.png)

可以看到 copy 方式导入速度是非常快的。关于 python 的 `copy_from` 方法请参考：
<http://initd.org/psycopg/docs/cursor.html>。

### 数据分析

在页面上点击分析指标，选择查询时间段，发送查询请求，后端收到请求后执行如下SQL查询：

```
SELECT time, data FROM t_unetanalysis_data
where uuid='xxx' and item_id=xxx and time>xxx and time<xxx;
```

例如，组织 id 为 50200021 的用户查询一个星期内 ip 的出量（item\_id为17），

```
SELECT time, data FROM t_unetanalysis_data
where uuid= '50200021' and item_id=17 and time > 1481472000 and time < 1482076800;
```

耗时平均为 260ms。

时间范围为 1 天的查询耗时平均为 120ms。

将查询到的数据返回给前端，前端解析数据，绘出图形，展示在页面上。

### 数据可视化

流量分析：

![image](/images/case2.4.png)

包量分析：

![image](/images/case2.5.png)

TCP延迟分析：

![image](/images/case2.6.png)
