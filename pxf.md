# PXF 扩展

在 5.17 及以上版本的 Udw 集群中默认安装了 PXF 扩展服务，Udw 集群可以通过 PXF 服务访问 HDFS, Hive, HBase 等外部数据，具体使用可以查询对应版本的 [GreenPlum PXF 官方文档](https://gpdb.docs.pivotal.io/5170/pxf/overview_pxf.html)。

使用 PXF 服务访问外部数据时，需要进行一些有关外部数据的配置，我们在控制台提供了配置上传的功能。如果需要访问 Hadoop 相关的外部数据，必须上传对应 Hadoop 集群的 `core-site.xml`, `hdfs-site.xml`, `mapred-site.xml`, `yarn-site.xml` 配置文件，如果还需要额外访问 Hive 或者 HBase 数据，则需要上传 `hive-site.xml` 或者 `hbase-site.xml` 配置文件。

因为配置文件中一般以域名/主机名表示各节点的访问地址，所以还需要额外上传包含 Hadoop 集群各节点的域名/主机名与 IP 对应关系的 `hosts` 文件，我们会将这个文件中的内容添加到 Udw 集群的 `hosts` 文件当中。(请尽量确保上传的 `hosts` 文件只包含集群各节点的 IP 信息，以免造在更新 Udw `hosts` 文件后造成错误)

上传配置之后，需要重启 PXF 服务使配置生效，控制台上提供了 PXF 服务的 停止/开启/重启 等操作功能。

## 配置 PXF 服务

在控制台 PXF 配置页面，有对应的文件列表与上传功能，点击 `上传` 并选择对应的 Hadoop 集群配置文件或者 `hosts` 文件，进行配置上传。

配置上传完成后，点击 `重启` 按钮让配置生效。

![image](/images/pxf/config-list.png)

## 创建 EXTENSION

连接 Greenplum 服务，创建 pxf EXTENSION：

    CREATE EXTENSION pxf;

授权给需要使用 pxf 外部表的用户：

    GRANT SELECT ON PROTOCOL pxf TO pan;
    GRANT INSERT ON PROTOCOL pxf TO pan;

## 读写 HDFS

在 Hadoop 集群中创建测试数据：

    $ hdfs dfs -mkdir -p /data/pxf_examples

    $ echo 'Prague,Jan,101,4875.33
    Rome,Mar,87,1557.39
    Bangalore,May,317,8936.99
    Beijing,Jul,411,11600.67' > /tmp/pxf_hdfs_simple.txt

    $ hdfs dfs -put /tmp/pxf_hdfs_simple.txt /data/pxf_examples/

连接 Greenplum 服务，创建外部表访问 HDFS 数据：

``` sql
CREATE EXTERNAL TABLE pxf_hdfs_textsimple(location text, month text, num_orders int, total_sales float8)
LOCATION ('pxf://data/pxf_examples/pxf_hdfs_simple.txt?PROFILE=hdfs:text')
FORMAT 'TEXT' (delimiter=E',');
```

查询外部表数据：

    postgres=# select * from pxf_hdfs_textsimple;
     location  | month | num_orders | total_sales
    -----------+-------+------------+-------------
     Prague    | Jan   |        101 |     4875.33
     Rome      | Mar   |         87 |     1557.39
     Bangalore | May   |        317 |     8936.99
     Beijing   | Jul   |        411 |    11600.67
    (4 rows)


创建可写外部表并插入数据：

``` sql
CREATE WRITABLE EXTERNAL TABLE pxf_hdfs_writabletbl (location text, month text, num_orders int, total_sales float8)
LOCATION ('pxf://data/pxf_examples/pxfwritable_hdfs_textsimple?PROFILE=hdfs:text&COMPRESSION_CODEC=org.apache.hadoop.io.compress.GzipCodec')
FORMAT 'TEXT' (delimiter=':');

INSERT INTO pxf_hdfs_writabletbl VALUES ( 'Cleveland', 'Oct', 3812, 96645.37 );
INSERT INTO pxf_hdfs_writabletbl VALUES ( 'Frankfurt', 'Mar', 777, 3956.98 );
```

在 HDFS 中查看数据：

    $ hdfs dfs -cat /data/pxf_examples/pxfwritable_hdfs_textsimple/* | zcat
    Cleveland:Oct:3812:96645.37
    Frankfurt:Mar:777:3956.98

> PXF 默认会以 postgres 这个用户名访问 HDFS，所以如果遇到权限错误，请将要访问/写入的 HDFS 目录授权给 postgres 用户

## 访问 Hive

准备 Hive 测试数据：

    $ echo 'Prague,Jan,101,4875.33
    Rome,Mar,87,1557.39
    Bangalore,May,317,8936.99
    Beijing,Jul,411,11600.67
    San Francisco,Sept,156,6846.34
    Paris,Nov,159,7134.56
    San Francisco,Jan,113,5397.89
    Prague,Dec,333,9894.77
    Bangalore,Jul,271,8320.55
    Beijing,Dec,100,4248.41
    ' > /tmp/pxf_hive_datafile.txt


创建 Hive 表并写入数据：

``` sql
CREATE TABLE sales_info (location string, month string,
number_of_orders int, total_sales double)
ROW FORMAT DELIMITED FIELDS TERMINATED BY ','
STORED AS textfile;

LOAD DATA LOCAL INPATH '/tmp/pxf_hive_datafile.txt'
INTO TABLE sales_info;
```

连接 Greenplum 服务，创建 PXF 外部表：

``` sql
CREATE EXTERNAL TABLE salesinfo_hiveprofile(location text, month text, num_orders int, total_sales float8)
LOCATION ('pxf://default.sales_info?PROFILE=Hive')
FORMAT 'custom' (formatter='pxfwritable_import');
```


通过 PXF 外部表查询 Hive 数据：


    postgres=# SELECT * FROM salesinfo_hiveprofile;
       location    | month | num_orders | total_sales
    ---------------+-------+------------+-------------
     Prague        | Jan   |        101 |     4875.33
     Rome          | Mar   |         87 |     1557.39
     Bangalore     | May   |        317 |     8936.99
     Beijing       | Jul   |        411 |    11600.67
     San Francisco | Sept  |        156 |     6846.34
     Paris         | Nov   |        159 |     7134.56
     San Francisco | Jan   |        113 |     5397.89
     Prague        | Dec   |        333 |     9894.77
     Bangalore     | Jul   |        271 |     8320.55
     Beijing       | Dec   |        100 |     4248.41
                   |       |            |
    (11 rows)


## 访问 HBase

如果需要支持 filter pushdown 特性，请根据 Udw 集群版本，下载对应的 `pxf-hbase-*.jar`，并复制到 HBase 集群每个节点的 `HBASE_CLASSPATH` 目录

[udw-6.2.1(pxf-hbase-5.10.1.jar)](http://udw.cn-bj.ufileos.com/pxf%2Fpxf-hbase-5.10.1.jar)
[udw-5.17(pxf-hbase-5.2.1.jar)](http://udw.cn-bj.ufileos.com/pxf%2Fpxf-hbase-5.2.1.jar)

准备测试数据，创建 HBase 表 `order_info`，该表有 2 个 column families：`product`, `shipping_info`：

```
create 'order_info', 'product', 'shipping_info'
```

在 `order_info` 表中插入测试数据：

```
put 'order_info', '1', 'product:name', 'tennis racquet'
put 'order_info', '1', 'product:location', 'out of stock'
put 'order_info', '1', 'shipping_info:state', 'CA'
put 'order_info', '1', 'shipping_info:zipcode', '12345'
put 'order_info', '2', 'product:name', 'soccer ball'
put 'order_info', '2', 'product:location', 'on floor'
put 'order_info', '2', 'shipping_info:state', 'CO'
put 'order_info', '2', 'shipping_info:zipcode', '56789'
put 'order_info', '3', 'product:name', 'snorkel set'
put 'order_info', '3', 'product:location', 'warehouse'
put 'order_info', '3', 'shipping_info:state', 'OH'
put 'order_info', '3', 'shipping_info:zipcode', '34567'
```

以直接映射的方式创建 PXF 外部表，以 HBase 的 `<column-family>:<column-qualifier>` 为外部表的列名：

``` sql
CREATE EXTERNAL TABLE orderinfo_hbase ("product:name" varchar, "shipping_info:zipcode" int)
LOCATION ('pxf://order_info?PROFILE=HBase')
FORMAT 'CUSTOM' (FORMATTER='pxfwritable_import');
```

通过 PXF 外部表访问 HBase 数据：

    postgres=# select * from orderinfo_hbase;
      product:name  | shipping_info:zipcode
    ----------------+-----------------------
     tennis racquet |                 12345
     soccer ball    |                 56789
     snorkel set    |                 34567
    (3 rows)
