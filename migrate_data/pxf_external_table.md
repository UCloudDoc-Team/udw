# 利用 hdfs 外部表迁移数据

利用外部 hdfs 作为中转，将原 greenplum 集群中的表数据迁移至目的 greenplum 集群，分为以下 4 步：

## 1. 在原 greenplum 集群中创建 hdfs pxf 可写外部表

假设需要迁移数据的表为 `src`，其结构如下所示：

    postgres=# \d+ src;
                                     Table "public.src"
       Column    |       Type       | Modifiers | Storage  | Stats target | Description
    -------------+------------------+-----------+----------+--------------+-------------
     location    | text             |           | extended |              |
     month       | text             |           | extended |              |
     num_orders  | integer          |           | plain    |              |
     total_sales | double precision |           | plain    |              |
    Distributed by: (location)

根据其表中字段以及定义，创建对应的 hdfs pxf 可写外部表：

``` sql
CREATE WRITABLE EXTERNAL TABLE pxf_hdfs_src (location text, month text, num_orders int, total_sales float8)
LOCATION ('pxf://data/udw/pxf_hdfs_src?PROFILE=hdfs:text&COMPRESSION_CODEC=org.apache.hadoop.io.compress.GzipCodec')
FORMAT 'TEXT' (delimiter=':');
```

## 2. 将原 greenplum 集群表数据写入 hdfs

把 hdfs 作为中转存储，将 `src` 表的数据写入 hdfs：

``` sql
insert into pxf_hdfs_src select * from src;
```

## 3. 在目的 greenplum 集群中创建 hdfs pxf 可读表

根据原 greenplum 集群中 pxf 外部表在目的集群创建相同结构与位置的 hdfs pxf 只读表：

``` sql
CREATE EXTERNAL TABLE pxf_hdfs_src(location text, month text, num_orders int, total_sales float8)
LOCATION ('pxf://data/udw/pxf_hdfs_src?PROFILE=hdfs:text&COMPRESSION_CODEC=org.apache.hadoop.io.compress.GzipCodec')
FORMAT 'TEXT' (delimiter=E':');
```

## 4. 从 hdfs 外部表中读取数据并写入目的 greenplum 集群

``` sql
insert into src select * from pxf_hdfs_src;
```

