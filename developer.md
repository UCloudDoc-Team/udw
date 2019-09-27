{{indexmenu_n>90}}

# 开发指南

## 1、连接数据库

udw 支持按照 postgresql 方式来访问 udw，可以支持 jdbc、odbc、php、python、psql 等方式来访问 udw。图形化的 pgAdmin、SQL Workbench/J 等工具

### 1.1 psql 客户端方式访问

下载 psql 客户端（或者通过控制台下载 udw 客户端）

```
yum install postgresql.i686 （32位系统）

yum install postgresql.x86\_64 (64位系统)

psql -h hostIP（或域名） –U username -d database -p port –W
```

* hostIP：udw master节点的ip或者域名
* username :数据库用户名
* database：数据库名称

### 1.2 udw 客户端方式访问

如果你选择是数据仓库类型是 greenplum、请用 greenplum 客户端、如果你选择的数据仓库类型请用 udpg 客户端。

1.1 udw（greenplum）客户端方式访问（以Centos为例）

1）下载greenplum客户端解压

```
wget http://udwclient.cn-bj.ufileos.com/greenplum-client.tar.gz

tar -zxvf greenplum-client.tar.gz
```

2）配置udw客户端

进入 greenplum-client 安装目录，编辑 `greenplum-client-path.sh`

修改 UDWHOME:

```
export UDWHOME= client安装目录（如/root/greenplum-client）
```

3） 使配置生效

在 `~/.bashrc` 中添加如下配置

```
source /data/greenplum-client/greenplum-client-path.sh
```

执行：

```
source ~/.bashrc
```

备注：`/data/greenplum-client` 是 greenplum-client 的安装路径

4） 连接数据库

```
psql -h hostIP（或域名） –U username -d database -p port –W
```

1.2 udw（udpg）客户端方式访问（以Centos为例）

1）下载 udw 客户端

```
wget http://udwclient.ufile.ucloud.cn/udw-client.tar

tar xvf udw-client.tar
```

2）配置 udw 客户端

进入 udw-client 安装目录，编辑 `udw-client-path.sh`

修改 UDWCLIENT:

```
export UDWCLIENT=client安装目录（如/root/udw-client）
```

3）使配置生效在 `~/.bashrc` 中添加如下配置

```
source /data/udw-client/udw-client-path.sh
```

执行：

```
source ~/.bashrc
```

备注：`/data/udw-client` 是 udw-client 的安装路径

4） 连接数据库

```
psql -h hostIP（或域名） –U username -d database -p port –W
```

### 1.3 SQL Workbench/J

工具访问udw的文档请参考：<https://static.ucloud.cn/7d32490688f9ddfca7b230c85158785b.pdf>

## 2、数据库管理

当你成功连接上数据库后，你可以创建你的第一个数据库（但这不是必须的，你也使用默认创建的数据库来作为你的业务数据库）。下面的操作以 psql 方式连接到 udw 为例。

### 2.1 创建数据库

    create database product;

### 2.2 查看所有数据库

“l”命令查看

![](/images/udw_developer.png)

或者通过下面sql查看

    select datname from pg\_database; (超级用户)

### 2.3 变更数据库

使用ALTER DATABASE命令，语法如下：

    ALTER DATABASE name [ [ WITH ] option [ ... ] ]
    where option can be:
         CONNECTION LIMIT connlimit
    ALTER DATABASE name SET parameter { TO | = } { value| DEFAULT }
    ALTER DATABASE name RESET parameter
    ALTER DATABASE name RENAME TO newname
    ALTER DATABASE name OWNER TO new_owner

### 2.4 删除数据库

    c template1 (切换到template1数据库)

    DROP DATABASE product;

## 3、模式管理

数据库模式(schema)是包含了一系列数据库对象（表，数据类型，自定义函数）集合的命名容器。一个数据库可以有多个模式。不同模式不共享命名空间。public 模式是在创建数据库之后就会默认创建的，每个用户都有权限在这个 schema 创建对象，如果不指定 schema 那么就会默认创建到这里。

创建一个模式:

    CREATE SCHEMA testSchema；

指定数据库的模式搜素路径：

    ALTER DATABASE product SET search_path To testSchema,public；

为指定用户指定模式搜素路径：

    ALTER ROLE roleName SET search_path To testSchema,public；

删除空模式：

    DROP SCHEMA testSchema；

删除非空模式：

    DROP SCHEMA testSchema CASCADE；

## 4、表格设计

udw 的表格创建类似于 postgresql，由于 udw 采用 mpp 数据，创建表格的时候可以选择不同的数据分布策略，不同的存储方式等等。创建表格的时候可以定义下面信息：

* 数据类型
* 表约束
* 数据分布策略
* 表存储模型
* 分区策略
* 外部表：udwfile、udwhdfs

下面分别根据上面的可选信息对表格设计进行分析。

### 4.1 数据类型

udw 的数据类型和 postgresql 基本一致，在选择数据类型的时候应该尽可能占用空间小，同时能够保证存储所有可能的数值并且最合理地表达数据。

使用字符型数据类型保存字符串，日期或者日期时间戳类型保存日期类型，数值类型来保存数值。

使用 VARCHAR 或者 TEXT 来保存文本类数据。不推荐使用 CHAR 类型保存文本类型。VARCHAR 或 TEXT 类型对于数据末尾的空白字符将原样保存和处理，但是 CHAR 类型不能满足这个需求。请参考 CREATE TABLE 命令了解更多相关信息。

使用 BIGINT 类型存储 INT 或者 SMALLINT 数值会浪费存储空间。如果数据随时间推移需要扩展，并且数据重新加载比较浪费时间，那么在开始的时候就应该考虑使用更大的数据类型。

### 4.2 表约束

udw 表格支持 postgresql 的表格约束，拥有 primary、unique 、check、not null、foreign 等约束，主键约束必须使用 hash 策略来分布表数据存储，不能在同一个表同时使用主键和唯一约束，并且指定了primary 和 unique 的列必须全部或者部分包含在分布键中。

创建表检查约束

    CREATE TABLE products(
        product_no integer,
        name text,
        price numeric CHECK (price > 0)
    );

创建非空约束

    CREATE TABLE products(
        product_no integer NOT NULL,
        name text NOT NULL,
        price numeric
    );

唯一约束：唯一约束确保存储在一张表中的一列或多列数据数据一定唯一。要使用唯一约束，表必须使用 Hash 分布策略，并且约束列必须和表的分布键对应的列一致（或者是超集）

    CREATE TABLE products(
        product_no integer UNIQUE,
        name text,
        price numeric
    ) DISTRIBUTED BY (product_no);

主键约束：主键约束是唯一约束和非空约束的组合。要使用主键约束，表必须使用 Hash 分布策略，并且约束列必须和表的分布键对应的列一致（或者是超集）。如果一张表指定了主键约束，分布键值默认会使用主键约束指定的列。

    CREATE TABLE products(
        product_no integer PRIMARY KEY,
        name text,
        price numeric
    ) DISTRIBUTED BY (product_no);

### 4.3 选择数据分布策略

UDW 表的记录有两种分布策略，分别是哈希分布（DISTRIBUTED BY(key)）和随机分布(DISTRIBUTED RANDOMLY)。如果不指定分布策略则默认按primary key或者第一个column 做哈希分布。

为了尽可能的并行处理数据，需要选择能够最大化地将数据均匀分布到所有计算节点的策略，比如选择 primary key；分布式处理中将会存在本地和分布式协作的操作，当不同的表使用相同的分布键的时候，大部分的排序、连接关联操作工作将会在本地完成，本地操作往往比分布式操作快很多，采用随机分布的策略无法享受到这个优势。

创建一个哈希分布的表：

    CREATE TABLE products (
        name varchar(40),
        prod_id int,
        supplier_id int
    ) DISTRIBUTED BY (prod_id);

创建一个随机分布的表：

    CREATE TABLE randomTable (
        things text,
        content text,
        etc text
    ) DISTRIBUTED RANDOMLY;

修改分布策略：

1）分区策略修改为随机分布：

    alter table test set with (reorganize=true) distributed randomly;

2）分区策略修改为按照id的hash分布：

    alter table test set with (reorganize=true) distributed by (id);

备注：更多关于分区策略的的使用可以通过命令行执行\\h create table 或者 \\h alter table 查看

### 4.4 表存储模型（heap表和appendonly表）

UDW 支持两种类型的表：堆表（heap table）和追加表（Appendonly table）。默认创建的是堆表。

堆表（heap table）是最普通的表形式，适合于较小、经常更新的数据存储方式。

追加表（Appendonly table）简称 ao 表，适合大表、updte 比较少的表。

创建一个堆表：

    CREATE TABLE heapTable(
        a int,
        b text
    ) DISTRIBUTED BY (a);

创建一个追加表（CREATE TABLE 命令的 WITH 子句来指定表存储模型）：

    CREATE TABLE aoTable(
        a int,
        b text
    ) WITH (appendonly=true) DISTRIBUTED BY (a);

### 4.5 表存储方式（行存储、列存储）

UDW支持行式存储、列式存储。

行存储的应用场景：

* 表数据在载入后经常 update;
* 表数据经常 insert；
* 查询中选择大部分的列；

列存储的应用场景：

列存储一般适用于宽表（即字段非常多的表）。在使用列存储时，同一个字段的数据连续保存在一个物理文件中，所以列存储的压缩率比普通压缩表的压缩率要高很多，另外在多数字段中筛选其中几个字段中，需要扫描的数据量很小，扫描速度比较快。因此，列存储尤其适合在宽表中对部分字段进行筛选的场景。注意：列存储的表必须是追加表（Appendonly table）。

创建一个行式存储的表

    CREATE TABLE rowTable(
        a int,
        b text
    ) WITH(appendonly=true, orientation=row) DISTRIBUTED BY (a);

创建一个列式存储的表

    CREATE TABLE colTable(
        a int,
        b text
    ) WITH(appendonly=true, orientation=column) DISTRIBUTED BY (a)

### 4.6 压缩表

UDW 压缩表必须是追加表。UDW 支持两种级别的压缩：表级别和字段级别。行式表和列式表对压缩的支持也不一样。

行式表支持表级别的压缩，支持的压缩算法有 ZLIB。

列式表支持表级别和字段级别的压缩，支持的压缩算法有 RLE\_TYPE，ZLIB。

RLE\_TYPE 的压缩级别 compresslevel 取值从1到4，级别越高压缩比越高。RLE\_TYPE适合于有大量重复的数据记录。

ZLIB 的压缩级别 compresslevel 取值从1到9，一般选择5已经足够了。

压缩表的应用场景：业务上对表进行更新和删除操作比较少，用 truncate＋delete 就可以实现业务逻辑。不经常对表进行加字段或修改字段类型，对 ao 表加字段比普通表慢很多。

创建一个使用 ZLIB 压缩的行压缩表：

    CREATE TABLE rowCompressTable(
        a int,
        b text
    ) WITH (appendonly=true,orientation=column,compresstype=ZLIB,compresslevel=5);

创建一个使用 RLE\_TYPE 压缩的列压缩表

    CREATE TABLE colCompressTable(
        c1 int,
        c2 char,
        c3 char
    ) WITH (appendonly=true, orientation=column, compresstype=RLE_TYPE,compresslevel=2);

### 4.7 外部表

外部表可以方便 udw 加载外部文件或者外部系统文件。详细使用请参考

外部表：[外部表并行加载数据到udw](http://udwclient.ufile.ucloud.com.cn/UDW%E5%A4%96%E9%83%A8%E8%A1%A8.pdf)

hdfs外部表：[创建hdfs外部表](http://udwclient.ufile.ucloud.com.cn/HDFS%E5%A4%96%E9%83%A8%E8%A1%A8.pdf)

ufile外部表：[创建ufile外部表](http://udwclient.cn-bj.ufileos.com/ufile%E5%AF%BC%E5%85%A5%E5%AF%BC%E5%87%BA%E6%95%B0%E6%8D%AE%E5%88%B0udw.pdf)

### 4.8 变更表

我们可以通过 `ALTER TABLE` 语句来更改一张表的定义，包括列的定义、数据分布策略、存储模型和分区结构。

给表中的某一列增加非空约束：

    ALTER TABLE test ALTER COLUMN street SET NOT NOT NULL;

改变表的数据分布策略

    ALTER TABLE test SET DISTRIBUTED BY (id);

其他更多可以通过执行`\h ALTER TABLE`查看帮助。

### 4.9 删除/清空表

删除表格：

    DROP TABLE test;

清空表数据：

    DELETE FROM test1;
    TRUNCATE test2;

## 5、加载数据

udw 提供了丰富的数据加载方式和工具：

* 用 postgresql 的 insert 和 copy 方式导入数据到 udw
* 用外部表的方式，把文件并行的导入到 udw
* 创建 hdfs 的外部表，导入导出数据到 hdfs
* 通过 sqoop 把 hdfs 中的数据导入到 udw
* 用 mysql2udw 把 mysql 中的数据导入到 udw
* 创建 ufile 的外部表、导入导出数据到 ufile
* 通过外部表导入 json 格式的数据

在导入大量的数据的时候我们建议不要使用 insert 一条条的导入数据、强烈建议使用 copy、udwfile 导入数据。

### 5.1 insert加载数据

我们可以通过insert插入数据到udw，语法如下所示：

    INSERT INTO 表名 \[ ( 字段 \[, ...\] ) \] { DEFAULT VALUES | VALUES ( { 表达式 | DEFAULT } \[, ...\] ) | 子查询 }

每次插入一条的效率会比较低、我们建议一次插入多条（500-5000条）数据。如果要加载的数据量比较大的话、强烈建议使用 copy 方式加载或者我们下面介绍的几种方式加载。如果您的数据已经在 udw 中，也可以通过 `insert into table1 select \* from table2` 这种方式加载数据。

### 5.2 copy加载数据

我们可以用copy快速加载文件数据到udw。具体语法如下：

    cat /data/test.dat | psql -h hostIP -U UserName -d DB -c "COPY employee from STDIN with CSV DELIMITER '|';"

* hostIP：udw访问id
* UserName ：访问数据的用户名
* DB：数据库名称
* employee：表名

### 5.3 外部表并行加载数据

外部表并行加载数据是利用 http 协议实现的一个文件服务器，用于创建 udw 的外部文件表。使用外部表并行加载数据可以让 udw 的每个子节点并行的加载数据、大大的加快数据导入 udw 的速度。在加载数据的时候我们可以先创建一个外部表，然后通过 `INSERT INTO <table> SELECT * FROM <external_table>`。这样就可以并行的加载文件中的数据。

使用方法请参考我们的文档：[外部表并行加载数据到udw](http://udwclient.ufile.ucloud.com.cn/UDW%E5%A4%96%E9%83%A8%E8%A1%A8.pdf)

### 5.4 从hdfs加载数据

为了方便 udw 和 hdfs 之间的数据导入和导出，我们提供个两种方案；

1.  用 sqoop 实现 hdfs 和 udw 直接的数据导入导出，使用方法请参考：[hdfs和hive中数据导入导出到udw](http://udwclient.ufile.ucloud.cn/hdfs和hive中数据导入导出到udw.pdf)
2.  创建 hdfs 外部表，使用方法请参考：[创建hdfs外部表](http://udwclient.ufile.ucloud.com.cn/HDFS%E5%A4%96%E9%83%A8%E8%A1%A8.pdf)

### 5.5 从mysql加载数据

为了方便 mysql 数据导入到 udw，我们提供了 mysql 导入数据到 udw 的工具 mysql2udw，使用方法请参考：[mysql数据导入到udw](http://udwclient.ufile.ucloud.cn/mysql数据导入到udw.pdf)

### 5.6 从oracle中导入数据

为了方便 oracle 数据导入 udw，我们提供了 oracle 导入数据到 udw 的工具 ora2udw,使用方法请参考：[oracle数据导入到udw](http://ora2udw.ufile.ucloud.com.cn/从Oracle导入数据到UDW.pdf)

### 5.7 从ufile加载数据

为了方便 ufile 数据导入到 udw，我们提供了 ufile 外部表，导入数据到 udw，使用方法请参考：[ufile数据导入到udw](http://udwclient.cn-bj.ufileos.com/ufile%E5%AF%BC%E5%85%A5%E5%AF%BC%E5%87%BA%E6%95%B0%E6%8D%AE%E5%88%B0udw.pdf)

## 6、分区表

分区表在逻辑上把一个大表切割成小表，分区表可以优化查询性能、在查询的时候只查询部分分区的内容。另外分区表可以很方便的让数据仓库把一些比较老的数据移出数据仓库。

目前udw支持的分区表类型有：

* range分区：把数据根据指定的范围进行分区，例如：时间范围、数值范围
* list分区：把数据按照一个list的值进行分区，例如：产品的种类、地区

使用分区表的场景：

* 数据表足够大：大表格是比较适合做分区的、如果你的表格有上亿行或者更多的的数据，可以通过分区把数据通过分区分为很多小的部分、从而提高性能。如果一个表只有几千行和几万行就不需要再做分区。
* 查询模式固定：例如你经常按照日期去查找表格数据、我们可以按照每月或者每天做分区；如果你需要按照地区去访问数据，我们可以按照地区去做分区。
* 数据仓库保留一个时间窗口的数据：例如您数据仓库需要保留一年的数据、如果按月做分区、可以通过分区很方便的删除最早的月份分区、把数据加载到最新的月份分区。
* 把数据分为几个均等的部分：通过一个分区标准把一个大表的数据划分为均等的分区，这样可以等倍的提高查询性能。

> 使用分区的时候请避免建立过多的分区，创建过多的分区可能会影响管理和维护作业，例如： 清理工作，节点恢复，集群扩展，查看磁盘使用情况等。

=== 6.1 创建分区表 ===

创建分区变需要注意以下问题：

* 确定分区策略：按照日期分区、按照数值分区、按照一个列表值分区
* 选择需要做分区的列
* 选择创建表格方式（heap表 ，append表、按列存储的表）

#### 6.1.1 按照日期创建分区表

日期划分的分区表使用一个日期或时间戳列做为分区键值列。可以按天或者按月进行分析。

您可以通过指定起始值（START），终止值（END）和增量子句（EVERY）指出分区的增量值，让 UDW 数据仓库来自动地生成分区。默认情况下，起始值总是包含的（闭区间），而终止值是排除的（开区间）。例如：

场景一：默认创建的分区表是heap表

    CREATE TABLE p_store_sales(
        id int,
        date date,
        prices decimal(7,2)
    ) DISTRIBUTED BY (id)
    PARTITION BY RANGE (date) (
        START (date '2016-01-01') INCLUSIVE
        END (date '2017-01-01') EXCLUSIVE
        EVERY (INTERVAL '1 day')
    );

场景二：创建appendonly的分区表

    CREATE TABLE p_store_sales(
        id int,
        date date,
        prices decimal(7,2)
    ) with (APPENDONLY=true ,compresslevel=5) DISTRIBUTED BY (id)
    PARTITION BY RANGE (date) (
        START (date '2016-01-01') INCLUSIVE
        END (date '2017-01-01') EXCLUSIVE
        EVERY (INTERVAL '1 day')
    );

场景三：创建列存储的分区表

    CREATE TABLE p_store_sales(
        id int,
        date date,
        prices decimal(7,2)
    ) with (APPENDONLY=true,ORIENTATION=column,compresslevel=5) DISTRIBUTED BY (id)
    PARTITION BY RANGE (date) (
        START (date '2016-01-01') INCLUSIVE
        END (date '2017-01-01') EXCLUSIVE
        EVERY (INTERVAL '1 day')
    );

场景四：创建带默认分区的分区表（一般情况不建议创建默认分区）

    CREATE TABLE p_store_sales(
        id int,
        date date,
        prices decimal(7,2)
    ) DISTRIBUTED BY (id)
    PARTITION BY RANGE (date) (
        START (date '2016-01-01') INCLUSIVE
        END (date '2017-01-01') EXCLUSIVE
        EVERY (INTERVAL '1 day')，
        DEFAULT PARTITION error
    );

如果输入的数据不满足分区的 CHECK 约束条件，并且没有创建默认分区，数据将被拒绝插入。
默认分区能够保证在输入数据不满足分区时，能够将数据插入到默认分区。

场景五：为每个分区指定独立的名：

    CREATE TABLE p_store_sales(
        id int,
        date date,
        prices decimal(7,2)
    ) DISTRIBUTED BY (id)
    PARTITION BY RANGE (date) (
      PARTITION Sep08 START (date '2016-09-01') INCLUSIVE ,
      PARTITION Oct08 START (date '2016-10-01') INCLUSIVE ,
      PARTITION Nov08 START (date '2016-11-01') INCLUSIVE ,
      PARTITION Dec08 START (date '2016-12-01') INCLUSIVE
      END (date '2017-01-01') EXCLUSIVE
    );

除了最后一个分区外，其他分区的终止值不用指出。

#### 6.1.2 按照数值范围创建分区表

使用数值范围的分区表，利用单独的数值类型列做为分区键值列。例如下面根据年龄范围做分区：

    CREATE TABLE p_members(
        id int,
        date date,
        age int,
        region char(1)
    ) DISTRIBUTED BY (id)
    PARTITION BY RANGE (age) (
        START (1) END (100) EVERY (1),
        DEFAULT PARTITION extra
    );

#### 6.1.3 按照列表值创建分区表

使用列表值进行分区的表可以选用任何支持等值比较的数据类型列做为分区键值列。对于列表值分区来说，您必须为每一个要创建的分区指定对应的列表值。例如下面根据地区进行分区：

    CREATE TABLE p_members(
        id int,
        date date,
        age int,
        region char(2)
    ) DISTRIBUTED BY (id)
    PARTITION BY LIST (region) (
        PARTITION shanghai VALUES ('SH'),
        PARTITION beijing VALUES ('BG'),
        DEFAULT PARTITION other
    );

### 6.2查看分区表信息

通过 pg\_partitions 视图，您可以查看分区表设计信息。下面示例可以查看 p\_store\_sales 表的分区设计信息：

    SELECT partitionboundary, partitiontablename, partitionname, partitionlevel, partitionrank
    FROM pg_partitions WHERE tablename='p_store_sales';

备注：

* pg\_partition ：跟踪分区表及其继承关系信息。
* pg\_partition\_templates ：创建子分区使用的子分区模版信息。
* pg\_partition\_columns ： 分区表分区键值信息。
* partitionrank：分区表的rank值，删除分区表的时候需要用这个值，详情请参考删除分区表

### 6.3 加载数据分区表

在创建了分区表结构后，父表里面是没有数据的。数据自动地存储到最底层的子分区中。

如果记录不满足任何子分区表的要求，插入将会被拒绝，数据加载都会失败。要避免不合要求的记录在加载时被拒绝导致的失败，可以在定义分区结构时，创建一个默认分区（DEFAULT）。任何不满足分区 CHECK 约束记录都会被加载到默认分区。

### 6.4 修改分区表

1.  通过修改父表来改变子表名称

        ALTER TABLE p_store_sales RENAME TO c_store_sales;

2.  通过修改指定分区名称，来更加便捷的识别子分区

        ALTER TABLE p_store_sales RENAME PARTITION FOR ('2016-01-01') TO part1;

### 6.5 增加分区/增加默认分区

增加分区：

您可以通过 ALTER TABLE 命令向已有的分区表中添加新的分区，例如：

    ALTER TABLE p_store_sales
    ADD PARTITION
        START (date '2017-02-01') INCLUSIVE
        END (date '2017-03-01') EXCLUSIVE
    with(APPENDONLY=true,ORIENTATION=column,compresslevel=5);

增加默认分区（一般不建议使用默认分区）：

    ALTER TABLE p_store_sales ADD DEFAULT PARTITION other;

如果输入的数据不满足分区的 CHECK 约束条件，并且没有创建默认分区，数据将被拒绝插入。默认分区能够保证在输入数据不满足分区时，能够将数据插入到默认分区。

如果分区表中包含默认分区，您必须通过分裂默认分区的方式来增加新的分区。在使用 INTO 子句时，需要将默认分区做为第二个分区名称。例如：

    ALTER TABLE p_store_sales
    SPLIT DEFAULT PARTITION
        START ('2016-01-01') INCLUSIVE
        END ('20016-02-01') EXCLUSIVE
    INTO (PARTITION new_part, default partition);

如果是按照列值创建的分区表，需要通过split default partition实现

    ALTER TABLE p_members
    SPLIT DEFAULT PARTITION at ('GZ') into (PARTITION guangzhou, DEFAULT PARTITION);

### 6.6 删除清空分区

删除分区

    ALTER TABLE p_store_sales DROP PARTITION FOR (RANK(1));

清空分区：

    ALTER TABLE p_store_sales TRUNCATE PARTITION FOR (RANK(1));

备注：RANK括号里面的1是分区的rank值、可以通过上述查看分区信息查看，增加或者减少分区都可能影响rank值。

### 6.7 把一个分区分为两个分区

使用 ALTER TABLE 命令来把一个分区分为两个分区

    ALTER TABLE p_store_sales SPLIT PARTITION FOR ('2016-01-01')
    AT ('2016-01-16')
    INTO (PARTITION part_001, PARTITION part_002);

如果您的分区表中包含默认分区，您必须通过分裂默认分区的方式来增加新的分区。在使用 INTO 子句时，需要将默认分区做为第二个分区名称。例如：

    ALTER TABLE p_store_sales SPLIT DEFAULT PARTITION
    START ('2016-01-01') INCLUSIVE
    END ('20016-02-01') EXCLUSIVE
    INTO (PARTITION new_part, default partition);

### 6.8 分区表索引管理

分区表对父表创建索引、分区表会自动增加索引，新增的分区也会自动添加索引。

例如：

    create index date_index on p_store_sales (date);

## 7、序列

通过使用序列，系统可以在新的纪录插入表中时，自动地按照自增方式分配一个唯一ID。使用序列一般就是为插入表中的纪录自动分配一个唯一标识符。您可以通过声明一个 SERIAL 类型的标识符列，该类型将会自动创建一个序列来分配 ID。

创建序列

    CREATE SEQUENCE myid START 0;

使用序列

    INSERT INTO test VALUES (nextval('myid'), 'test');

修改序列

    ALTER SEQUENCE myid RESTART WITH 88;

删除序列

    DROP SEQUENCE myid;

## 8、索引

udw 支持B-tree、位图索引（bitmap）

默认创建的是 B-tree 索引；唯一索引必须包含在分布键中（可以是全部或者部分列，在第1个索引中可以部分列，之后必须全部列），唯一索引不支持 ao 表，唯一索引不会跨越分区起作用，只是针对独立的分区；位图索引比普通索引占用更小的空间，位图索引不应在经常更新的表中使用。

btree索引

    CREATE UNIQUE INDEX test_index ON t1 (id);

位图索引

    CREATE INDEX test_bmp_index ON t2 USING bitmap(id);

更改索引：

    Alter INDEX name RENAME TO new_name;

查看表的索引信息

    select * from pg_indexes where tablename='t1';

检查索引使用

查看 EXPLAIN 输出中出现的提示：

    EXPLAIN select id, name from t1 where id = 100;

如有 Index Scan 或者 Bitmap Heap Scan 或者 Bitmap Index Scan, 则使用了索引。

删除索引：

    drop index test_index;

关于建索引的一些建议：

* 不要在频繁更新的字段上建索引
* 索引列通常用来做join
* 批量导入数据需先删除索引，等数据导完后再重建，这样会更快
* 索引列经常被频繁使用在where语句中

## 9、 ANALYZE/VACUUM

ANALYZE：收集数据库的统计信息。

    ANALYZE [VERBOSE] [ROOTPARTITION [ALL] ]
    [table [ (column [, ...] ) ]]

VACUUM：数据库磁盘垃圾回收并收集统计信息。

    VACUUM [FULL] [FREEZE] [VERBOSE] [table]
    VACUUM [FULL] [FREEZE] [VERBOSE] ANALYZE
    [table [(column [, ...] )]]

## 10、常用SQL大全

1.  psql客户端常用

\\h 获取SQL命令的帮助

\\? 获取psql命令的帮助

\\q 退出

1.  一般选项

\\c \[数据库名\]－\[用户名\] 连接到新数据库

\\cd \[目录名\] 改变当前的工作目录

\\encoding \[编码\] 显示或设置客户端编码

\\h \[名字\] SQL命令的语法帮助

\\set \[名字 ［值］\] 设置内部变量

\\timing 查询计时开关切换（默认关闭）

\\unset 名字 取消（删除）内部变量

1.  查询缓冲区选项

\\e \[文件名\] 用一个外部编辑器编辑当前查询缓冲区或文件

\\g［文件名］向服务器发送SQL命令

\\p 显示当前查询缓冲区的内容

\\r 重置 (清理) 查询缓冲区

\\s \[文件名\] 打印历史或者将其保存到文件

\\w \[文件名\] 将查询缓冲区写出到文件

1.  输入／输出选项

\\echo \[字串\] 向标准输出写出文本 /i 文件名 执行来自文件的命令 

\\o \[文件名\] 向文件或者 |管道 发送所有查询结果 

1.  信息选项

\\d \[名字\] 描述表, 索引, 序列, 或者视图

\\d{t|i|s|v|S} \[模式\] (加 "+" 获取更多信息)   列出表/索引/序列/视图/系统表 

\\da \[模式\] 列出聚集函数 

\\db \[模式\] 列出表空间 (加 "+" 获取更多的信息) 

\\dc \[模式\] 列出编码转换 

\\dC 列出类型转换 

\\dd \[模式\] 显示目标的注释

\\df \[模式\] 列出函数 (加 "+" 获取更多的信息) 

\\dg \[模式\] 列出组 

\\dn \[模式\] 列出模式 (加 "+" 获取更多的信息) 

\\do \[名字\] 列出操作符 

\\dl 列出大对象, 和 lo\_list 一样 

\\dp \[模式\] 列出表, 视图, 序列的访问权限 

\\dT \[模式\] 列出数据类型 (加 "+" 获取更多的信息) 

\\du \[模式\] 列出用户 

\\l 列出所有数据库 (加 "+" 获取更多的信息) 

\\z \[模式\] 列出表, 视图, 序列的访问权限 (和 dp 一样)

1.  格式选项 

\\a 在非对齐和对齐的输出模式之间切换 

\\C \[字串\] 设置表标题, 如果参数空则取消标题 

\\f \[字串\] 为非对齐查询输出显示或设置域分隔符 

\\H 在 HTML 输出模式之间切换 (当前是 关闭) 

\\pset 变量 \[值\]   设置表的输出选项 

\\t 只显示行 (当前是 关闭) 

\\T \[字串\] 设置 HTML \<表\> 标记属性, 如果没有参数就取消设置

\\x 在扩展输出之间切换 (目前是 关闭) 

## 12、常用SQL命令

命令：ABORT

描述: 终止当前事务 

语法:

    ABORT [ WORK | TRANSACTION ]  

命令: ALTER AGGREGATE 

描述: 改变一个聚集函数的定义

语法：

    ALTER AGGREGATE 名字 ( 类型 ) RENAME TO 新名字

命令: ALTER DATABASE 

描述: 改变一个数据库 

语法:

    ALTER DATABASE 名字 SET 参数 { TO | = } { 值 | DEFAULT } 
    ALTER DATABASE 名字 RESET 参数 
    ALTER DATABASE 名字 RENAME TO 新名字 
    ALTER DATABASE 名字 OWNER TO 新属主 

命令: ALTER FUNCTION 

描述: 改变一个函数的定义 

语法:

    ALTER FUNCTION 名字 ( [ 类型 [, ...] ] ) RENAME TO 新名字 
    ALTER FUNCTION 名字 ( [ 类型 [, ...] ] ) OWNER TO 新属主 

命令: ALTER GROUP 

描述: 改变一个用户组 

语法:

    ALTER GROUP 组名称 ADD USER 用户名称 [, ... ] 
    ALTER GROUP 组名称 DROP USER 用户名称 [, ... ] 
    ALTER GROUP 组名称 RENAME TO 新名称 

命令: ALTER INDEX 

描述: 改变一个索引的定义 

语法:

    ALTER INDEX name RENAME TO new_name
    ALTER INDEX name SET TABLESPACE tablespace_name
    ALTER INDEX name SET (storage_parameter = value )
    ALTER INDEX name RESET (storage_parameter [, ... ])  

命令: ALTER SCHEMA 

描述: 改变一个模式的定义 

语法:

    ALTER SCHEMA 名字 RENAME TO 新名字 
    ALTER SCHEMA 名字 OWNER TO 新属主

  命令: ALTER SEQUENCE 

描述: 改变一个序列生成器的定义 

语法:

```
ALTER SEQUENCE 名字 [ INCREMENT [ BY ] 递增 ]  [ MINVALUE 最小值 | NO MINVALUE ] [ MAXVALUE 最大值 | NO MAXVALUE ] [ RESTART [ WITH ] 开始 ] [ CACHE 缓存 ] [ [ NO ] CYCLE ]  [OWNED BY {表名.列名 | NONE}] ALTER SEQUENCE name SET SCHEMA new_schema
```

命令: ALTER TYPE 

描述: 改变一个类型的定义语法: 

    ALTER TYPE 名字 OWNER TO 新属主 

命令: ALTER USER 

描述: 改变一个数据库用户 

语法: 

    ALTER USER name RENAME TO newname 
    ALTER USER name SET parameter { TO | = } { value | DEFAULT } 
    ALTER USER name RESET parameter 
    ALTER USER name [ [ WITH ] option [ ... ] ]  where option can be:   CREATEDB | NOCREATEDB    | CREATEUSER | NOCREATEUSER    | [ ENCRYPTED | UNENCRYPTED ] PASSWORD 'password'    | VALID UNTIL 'abstime' 

命令: ANALYZE 

描述: 收集关于数据库的统计信息

语法: 

```
ANALYZE [VERBOSE] [ROOTPARTITION [ALL] ] [table [ (column [, ...] ) ]]
```

命令: BEGIN 

描述: 开始一个事务块 

语法:

    BEGIN [ WORK | TRANSACTION ] [ 事务模式]  [ READ WRITE | READ ONLY  ]
    事务模式为下面之一:
    ISOLATION LEVEL { SERIALIZABLE | READ COMMITTED | READ UNCOMMITTED } 

命令: CLOSE 

描述: 关闭一个游标 

语法: CLOSE 名字 

命令: COMMIT 

描述: 提交当前事务 

语法: COMMIT \[ WORK | TRANSACTION \] 

命令: COPY 

描述: 在一个文件和一个表之间拷贝数据 

语法:

    COPY table [(column [, ...])] FROM {'file' | STDIN} [ [WITH] [OIDS] [HEADER] [DELIMITER [ AS ] 'delimiter'] [NULL [ AS ] 'null string'] [ESCAPE [ AS ] 'escape' | 'OFF'] [NEWLINE [ AS ] 'LF' | 'CR' | 'CRLF'] [CSV [QUOTE [ AS ] 'quote'] [FORCE NOT NULL column [, ...]] [FILL MISSING FIELDS] [[LOG ERRORS [INTO error_table] [KEEP] SEGMENT REJECT LIMIT count [ROWS | PERCENT] ] COPY {table [(column [, ...])] | (query)} TO {'file' | STDOUT} [ [WITH] [OIDS] [HEADER] [DELIMITER [ AS ] 'delimiter'] [NULL [ AS ] 'null string'] [ESCAPE [ AS ] 'escape' | 'OFF'] [CSV [QUOTE [ AS ] 'quote']  [FORCE QUOTE column [, ...]] ] [IGNORE EXTERNAL PARTITIONS ]

命令: CREATE AGGREGATE 

描述: 定义一个新的聚集函数 

语法: 

```
CREATE [ORDERED] AGGREGATE name (input_data_type [ , ... ]) ( SFUNC = sfunc, STYPE = state_data_type [, PREFUNC = prefunc] [, FINALFUNC = ffunc] [, INITCOND = initial_condition] [, SORTOP = sort_operator] )
```

命令: CREATE CAST 

描述: 定义一个新的类型转换 

语法:

    CREATE CAST (源类型 AS 目标类型)    WITH FUNCTION 函数名 (参数类型)    [ AS ASSIGNMENT | AS IMPLICIT ] CREATE CAST (源类型 AS 目标类型)    WITHOUT FUNCTION    [ AS ASSIGNMENT | AS IMPLICIT ] 

命令: CREATE DATABASE 

描述: 创建一个新的数据库 

语法: 

    CREATE DATABASE 数据库名称    [ [ WITH ] [ OWNER [=] 数据库属主 ]    [ TEMPLATE [=] 模板 ]    [ ENCODING [=] 编码 ]    [ TABLESPACE [=] 表空间 ] ]  

命令: CREATE FUNCTION 

描述: 定义一个新的函数 

语法: 

    CREATE [ OR REPLACE ] FUNCTION 名字 ( [ [ 参数名字 ] 参数类型 [, ...] ] )   RETURNS 返回类型  { LANGUAGE 语言名称    | IMMUTABLE | STABLE | VOLATILE    | CALLED ON NULL INPUT | RETURNS NULL ON NULL INPUT | STRICT    | [ EXTERNAL ] SECURITY INVOKER | [ EXTERNAL ] SECURITY DEFINER    | AS 'definition'    | AS 'obj_file', 'link_symbol'  } …   [ WITH ( attribute [, ...] ) ] 

命令: CREATE GROUP 

描述: 定义一个新的用户组 

语法:

```
CREATE GROUP 组名 [ [ WITH ] option [ ... ] ]  option 可以为:   SUPERUSER | NOSUPERUSER | CREATEDB | NOCREATEDB | CREATEROLE | NOCREATEROLE | CREATEUSER | NOCREATEUSER | INHERIT | NOINHERIT | LOGIN | NOLOGIN | [ ENCRYPTED | UNENCRYPTED ] PASSWORD 'password' | VALID UNTIL 'timestamp' | IN ROLE rolename [, ...] | IN GROUP rolename [, ...] | ROLE rolename [, ...] | ADMIN rolename [, ...] | USER rolename [, ...] | SYSID uid
```

命令: CREATE INDEX 

描述: 定义一个新的索引 

语法:

```
CREATE [ UNIQUE ] INDEX 索引名称 ON 表名 [ USING method ]    ( { column | ( expression ) } [ opclass ] [, ...] )    [ TABLESPACE tablespace ]    [ WHERE predicate ] 
```

命令: CREATE RULE 

描述: 定义一个新的重写规则 

语法: 

```
CREATE [ OR REPLACE ] RULE 名字 AS ON 事件    TO 表 [ WHERE 条件 ]    DO [ ALSO | INSTEAD ] { NOTHING | 命令 | ( 命令 ; 命令 ... ) }  
```

命令: CREATE SCHEMA 

描述: 定义一个新的模式 

语法: 

    CREATE SCHEMA 模式名称 [ AUTHORIZATION 用户名称 ] [ 模式元素 [ ... ] ] 
    CREATE SCHEMA AUTHORIZATION 用户名称 [ 模式元素 [ ... ] ] 

命令: CREATE SEQUENCE 

描述: 定义一个新的序列生成器 

语法:

```
CREATE [ TEMPORARY | TEMP ] SEQUENCE name [ INCREMENT [ BY ] increment ]   [ MINVALUE minvalue | NO MINVALUE ] [ MAXVALUE maxvalue | NO MAXVALUE ]  [ START [ WITH ] start ] [ CACHE cache ] [ [ NO ] CYCLE ]  [OWNED BY { table.column | NONE }]
```

命令: CREATE TABLE 

描述: 定义一个新的表 

语法: 

    CREATE [ [ GLOBAL | LOCAL ] { TEMPORARY | TEMP } ] TABLE table_name (  { column_name data_type [ DEFAULT default_expr ] [ column_constraint [ ... ] ]    | table_constraint   | LIKE parent_table [ { INCLUDING | EXCLUDING } DEFAULTS ] } [, ... ] )  [ INHERITS ( parent_table [, ... ] ) ]  [ WITH OIDS | WITHOUT OIDS ]  [ ON COMMIT { PRESERVE ROWS | DELETE ROWS | DROP } ]  [ TABLESPACE tablespace ]  where column_constraint is:  [ CONSTRAINT constraint_name ]  { NOT NULL | NULL | UNIQUE [ USING INDEX TABLESPACE tablespace ] |  PRIMARY KEY [ USING INDEX TABLESPACE tablespace ] | CHECK (expression) |  REFERENCES reftable [ ( refcolumn ) ] [ MATCH FULL | MATCH PARTIAL | MATCH SIMPLE ]    [ ON DELETE action ] [ ON UPDATE action ] }  [ DEFERRABLE | NOT DEFERRABLE ] [ INITIALLY DEFERRED | INITIALLY  IMMEDIATE ]  and table_constraint is:  [ CONSTRAINT constraint_name ]  { UNIQUE ( column_name [, ... ] ) [ USING INDEX TABLESPACE tablespace ] |  PRIMARY KEY ( column_name [, ... ] ) [ USING INDEX TABLESPACE tablespace ] |  CHECK ( expression ) | FOREIGN KEY ( column_name [, ... ] ) REFERENCES reftable [ ( refcolumn [, ... ] ) ]    [ MATCH FULL | MATCH PARTIAL | MATCH SIMPLE ] [ ON DELETE action ] [ ON UPDATE action ] } [ DEFERRABLE | NOT DEFERRABLE ] [ INITIALLY DEFERRED | INITIALLY  IMMEDIATE ] 

命令: CREATE TABLE AS 

描述: 以一个查询的结果定义一个新的表 

语法: 

    CREATE [ [ GLOBAL | LOCAL ] { TEMPORARY | TEMP } ] TABLE 表名字 [ (字段名字 [, ...] ) ] [ [ WITH | WITHOUT ] OIDS ]  AS query 

命令: CREATE TYPE 

描述: 定义一个新的数据类型 

语法:

```
CREATE TYPE name AS ( attribute_name data_type [, ... ] ) CREATE TYPE name ( INPUT = input_function, OUTPUT = output_function [, RECEIVE = receive_function] [, SEND = send_function] [, INTERNALLENGTH = {internallength | VARIABLE}] [, PASSEDBYVALUE] [, ALIGNMENT = alignment] [, STORAGE = storage] [, DEFAULT = default] [, ELEMENT = element] [, DELIMITER = delimiter] ) CREATE TYPE name
```

命令: CREATE USER 

描述: 定义一个新的数据库用户帐户 

语法: 

```
CREATE USER name [ [ WITH ] option [ ... ] ]  where option can be:  SUPERUSER | NOSUPERUSER | CREATEDB | NOCREATEDB | CREATEROLE | NOCREATEROLE | CREATEUSER | NOCREATEUSER | INHERIT | NOINHERIT | LOGIN | NOLOGIN | [ ENCRYPTED | UNENCRYPTED ] PASSWORD 'password' | VALID UNTIL 'timestamp' | IN ROLE rolename [, ...] | IN GROUP rolename [, ...] | ROLE rolename [, ...] | ADMIN rolename [, ...] | USER rolename [, ...] | SYSID uid | RESOURCE QUEUE queue_name
```

命令: CREATE VIEW 描述: 定义一个新的视图 语法: 

```
CREATE [OR REPLACE] [TEMP | TEMPORARY] VIEW name [ ( column_name [, ...] ) ] AS query
```

 

命令: DECLARE 

描述: 定义一个游标 

语法: 

```
DECLARE name [BINARY] [INSENSITIVE] [NO SCROLL] CURSOR [{WITH | WITHOUT} HOLD] FOR query [FOR READ ONLY]
```

命令: DELETE 

描述: 删除一个表的记录 

语法: 

    DELETE FROM [ ONLY ] 表 [ WHERE 条件] 

命令: DROP AGGREGATE 

描述: 删除一个聚集函数 

语法: 

    DROP AGGREGATE 名字 ( 类型 ) [ CASCADE | RESTRICT ] 

命令: DROP CAST 

描述: 删除一个类型转换 

语法:

    DROP CAST (源类型 AS 目标类型) [ CASCADE | RESTRICT ] 

命令: DROP DATABASE 

描述: 删除一个数据库 

语法: 

    DROP DATABASE 名字 

命令: DROP FUNCTION 

描述: 删除一个函数 

语法: 

    DROP FUNCTION 名字 ( [ 类型 [, ...] ] ) [ CASCADE | RESTRICT ] 

命令: DROP GROUP 

描述: 删除一个用户组 

语法: 

    DROP GROUP 名字 

命令: DROP INDEX 

描述: 删除一个索引 

语法:

    DROP INDEX 名字 [, ...] [ CASCADE | RESTRICT ] 

命令: DROP RULE 

描述: 删除一个重写规则 

语法: 

    DROP RULE 名字 ON 关系 [ CASCADE | RESTRICT ] 

命令: DROP SCHEMA 

描述: 删除一个模式 

语法: 

    DROP SCHEMA 名字 [, ...] [ CASCADE | RESTRICT ] 

命令: DROP SEQUENCE 

描述: 删除一个序列 

语法:

    DROP SEQUENCE 名字 [, ...] [ CASCADE | RESTRICT ] 

命令: DROP TABLE 

描述: 删除一个表

语法: 

    DROP TABLE 名字 [, ...] [ CASCADE | RESTRICT ] 

命令: DROP TYPE 

描述: 删除一个数据类型 

语法: 

    DROP TYPE 名字 [, ...] [ CASCADE | RESTRICT ] 

命令: DROP USER 

描述: 删除一个数据库用户帐户 

语法: 

    DROP USER 名字 

命令: DROP VIEW 

描述: 删除一个视图 

语法:

    DROP VIEW 名字 [, ...] [ CASCADE | RESTRICT ] 

命令: END 

描述: 提交当前事务 

语法: 

    END [ WORK | TRANSACTION ] 

命令: EXECUTE 

描述: 执行一个准备好的语句 

语法: 

    EXECUTE 规划名称 [ (参数 [, ...] ) ] 

命令: EXPLAIN 

描述: 显示语句的执行规划 

语法: 

    EXPLAIN [ ANALYZE ] [ VERBOSE ] 语句 

命令: FETCH 

描述: 恢复来自一个使用游标查询的行 

语法:

```
FETCH [ direction { FROM | IN } ] cursorname  direction 可以为空或下面的一种:  NEXT | FIRST | LAST | ABSOLUTE count  | RELATIVE count | count | ALL | FORWARD |  FORWARD count  | FORWARD ALL
```

命令: GRANT 

描述: 定义访问权限 

语法:

```
IGRAN { {SELECT | INSERT | UPDATE | DELETE | REFERENCES | TRIGGER | TRUNCATE } [,...] | ALL [PRIVILEGES] } ON [TABLE] tablename [, ...] TO {rolename | PUBLIC} [, ...] [WITH GRANT OPTION] GRANT { {USAGE | SELECT | UPDATE} [,...] | ALL [PRIVILEGES] } ON SEQUENCE sequencename [, ...] TO { rolename | PUBLIC } [, ...] [WITH GRANT OPTION] GRANT { {CREATE | CONNECT | TEMPORARY | TEMP} [,...] | ALL [PRIVILEGES] } ON DATABASE dbname [, ...] TO {rolename | PUBLIC} [, ...] [WITH GRANT OPTION] GRANT { EXECUTE | ALL [PRIVILEGES] } ON FUNCTION funcname ( [ [argmode] [argname] argtype [, ...] ] ) [, ...] TO {rolename | PUBLIC} [, ...] [WITH GRANT OPTION] GRANT { USAGE | ALL [PRIVILEGES] } ON LANGUAGE langname [, ...] TO {rolename | PUBLIC} [, ...] [WITH GRANT OPTION] GRANT { {CREATE | USAGE} [,...] | ALL [PRIVILEGES] } ON SCHEMA schemaname [, ...] TO {rolename | PUBLIC} [, ...] [WITH GRANT OPTION] GRANT { CREATE | ALL [PRIVILEGES] } ON TABLESPACE tablespacename [, ...] TO {rolename | PUBLIC} [, ...] [WITH GRANT OPTION] GRANT parent_role [, ...] TO member_role [, ...] [WITH ADMIN OPTION] GRANT { SELECT | INSERT | ALL [PRIVILEGES] } ON PROTOCOL protocolname TO username
```

命令: INSERT 

描述: 在一个表中插入记录 

语法:

    INSERT INTO 表名 [ ( 字段 [, ...] ) ] { DEFAULT VALUES | VALUES ( { 表达式 | DEFAULT } [, ...] ) | 子查询 } 

命令: LOCK 

描述: 锁定一个表 

语法:

    LOCK [ TABLE ] 名字 [, ...] [ IN lockmode MODE ] [ NOWAIT ] 
    lockmode 可以是下面的一种: 
    ACCESS SHARE | ROW SHARE | ROW EXCLUSIVE | SHARE UPDATE EXCLUSIVE | SHARE | SHARE ROW EXCLUSIVE | EXCLUSIVE | ACCESS EXCLUSIVE

命令: PREPARE 

描述: 为执行准备一条语句 

语法:

    PREPARE 规划名称 [ (数据类型 [, ...] ) ] AS 语句 

命令: REINDEX 

描述: 重建索引 

语法: 

    REINDEX { DATABASE | TABLE | INDEX } 名字 [ FORCE ] 

命令: RELEASE SAVEPOINT 

描述: 删除一个以前定义的 savepoint 

语法: 

    RELEASE [ SAVEPOINT ] savepoint_name 

命令: RESET 

描述: 恢复运行时参数值为默认值 

语法: 

    RESET 名字 
    RESET ALL 

命令: REVOKE  描述: 删除访问权限  语法:

    REVOKE [GRANT OPTION FOR] { {SELECT | INSERT | UPDATE | DELETE | REFERENCES | TRIGGER | TRUNCATE } [,...] | ALL [PRIVILEGES] } ON [TABLE] tablename [, ...] FROM {rolename | PUBLIC} [, ...] [CASCADE | RESTRICT] REVOKE [GRANT OPTION FOR] { {USAGE | SELECT | UPDATE} [,...] | ALL [PRIVILEGES] } ON SEQUENCE sequencename [, ...] FROM { rolename | PUBLIC } [, ...] [CASCADE | RESTRICT] REVOKE [GRANT OPTION FOR] { {CREATE | CONNECT | TEMPORARY | TEMP} [,...] | ALL [PRIVILEGES] } ON DATABASE dbname [, ...] FROM {rolename | PUBLIC} [, ...] [CASCADE | RESTRICT] REVOKE [GRANT OPTION FOR] {EXECUTE | ALL [PRIVILEGES]} ON FUNCTION funcname ( [[argmode] [argname] argtype [, ...]] ) [, ...] FROM {rolename | PUBLIC} [, ...] [CASCADE | RESTRICT] REVOKE [GRANT OPTION FOR] {USAGE | ALL [PRIVILEGES]} ON LANGUAGE langname [, ...] FROM {rolename | PUBLIC} [, ...] [ CASCADE | RESTRICT ] REVOKE [GRANT OPTION FOR] { {CREATE | USAGE} [,...] | ALL [PRIVILEGES] } ON SCHEMA schemaname [, ...] FROM {rolename | PUBLIC} [, ...] [CASCADE | RESTRICT] REVOKE [GRANT OPTION FOR] { CREATE | ALL [PRIVILEGES] } ON TABLESPACE tablespacename [, ...] FROM { rolename | PUBLIC } [, ...] [CASCADE | RESTRICT] REVOKE [ADMIN OPTION FOR] parent_role [, ...] FROM member_role [, ...]  [CASCADE | RESTRICT]

命令: ROLLBACK 

描述: 终止当前事务 

语法:

    ROLLBACK [ WORK | TRANSACTION ] 

命令: ROLLBACK TO SAVEPOINT 

描述: 回滚到一个 savepoint 

语法: 

    ROLLBACK [ WORK | TRANSACTION ] TO [ SAVEPOINT ] savepoint_name 

  命令: SAVEPOINT 

描述: 在当前事物中定义一个新的 savepoint 

语法: 

    SAVEPOINT savepoint_name

命令: SELECT  

描述: 从表或视图中查询记录

语法:

```
SELECT [ALL | DISTINCT [ON (expression [, ...])]] * | expression [[AS] output_name] [, ...] [FROM from_item [, ...]] [WHERE condition] [GROUP BY grouping_element [, ...]] [HAVING condition [, ...]] [WINDOW window_name AS (window_specification)] [{UNION | INTERSECT | EXCEPT} [ALL] select] [ORDER BY expression [ASC | DESC | USING operator] [, ...]] [LIMIT {count | ALL}] [OFFSET start] [FOR {UPDATE | SHARE} [OF table_name [, ...]] [NOWAIT] [...]] where grouping_element can be one of: () expression ROLLUP (expression [,...]) CUBE (expression [,...]) GROUPING SETS ((grouping_element [, ...])) where window_specification can be: [window_name] [PARTITION BY expression [, ...]] [ORDER BY expression [ASC | DESC | USING operator] [, ...] [{RANGE | ROWS} { UNBOUNDED PRECEDING | expression PRECEDING | CURRENT ROW | BETWEEN window_frame_bound AND window_frame_bound }]] where window_frame_bound can be one of: UNBOUNDED PRECEDING expression PRECEDING CURRENT ROW expression FOLLOWING UNBOUNDED FOLLOWING where from_item can be one of: [ONLY] table_name [[AS] alias [( column_alias [, ...] )]] (select) [AS] alias [( column_alias [, ...] )] function_name ( [argument [, ...]] ) [AS] alias [( column_alias [, ...] | column_definition [, ...] )] function_name ( [argument [, ...]] ) AS ( column_definition [, ...] ) from_item [NATURAL] join_type from_item [ON join_condition | USING ( join_column [, ...] )]
```

命令: SELECT INTO 

描述: 以一个查询的结果定义一个新的表 

语法:

```
SELECT [ALL | DISTINCT [ON ( expression [, ...] )]] * | expression [AS output_name] [, ...] INTO [TEMPORARY | TEMP] [TABLE] new_table [FROM from_item [, ...]] [WHERE condition] [GROUP BY expression [, ...]] [HAVING condition [, ...]] [{UNION | INTERSECT | EXCEPT} [ALL] select] [ORDER BY expression [ASC | DESC | USING operator] [, ...]] [LIMIT {count | ALL}] [OFFSET start] [FOR {UPDATE | SHARE} [OF table_name [, ...]] [NOWAIT] [...]]
```

命令: SET 

描述: 改变一个运行时参数 

语法: 

```
SET [SESSION | LOCAL] configuration_parameter {TO | =} value | 'value' | DEFAULT}
SET [SESSION | LOCAL] TIME ZONE {timezone | LOCAL | DEFAULT}
```

命令: SET TRANSACTION 

描述: 设置当前事务的属性 

语法:

```
SET TRANSACTION [transaction_mode] [READ ONLY | READ WRITE] SET SESSION CHARACTERISTICS AS TRANSACTION transaction_mode [READ ONLY | READ WRITE]
事务模式为其中之一:
ISOLATION LEVEL {SERIALIZABLE | READ COMMITTED | READ UNCOMMITTED}
```

命令: SHOW 

描述: 显示运行时参数值 

语法:

```
SHOW 参数名
SHOW ALL
```

命令: START TRANSACTION 

描述: 开始一个事务块

语法:

    START TRANSACTION [ 事务模式 [, ...] ]  
    事务模式为下面之一: 
    ISOLATION LEVEL { SERIALIZABLE | READ COMMITTED | READ UNCOMMITTED } READ WRITE | READ ONLY 

命令: TRUNCATE

描述: 清空一个表

语法:

```
TRUNCATE [TABLE] name [, ...] [CASCADE | RESTRICT]
```

命令: UPDATE 

描述: 更新表的记录

语法:

```
UPDATE [ONLY] table [[AS] alias] SET {column = {expression | DEFAULT} | (column [, ...]) = ({expression | DEFAULT} [, ...])} [, ...] [FROM fromlist] [WHERE condition | WHERE CURRENT OF cursor_name ]
```

命令: VACUUM

描述: 垃圾回收和可选择分析一个数据库

语法:

```
VACUUM [FULL] [FREEZE] [VERBOSE] [table]
VACUUM [FULL] [FREEZE] [VERBOSE] ANALYZE [table [(column [, ...] )]]
```

命令: VALUES 

描述: 计算值表达式的值

语法:

```
VALUES ( expression [, ...] ) [, ...] [ORDER BY sort_expression [ASC | DESC | USING operator] [, ...]] [LIMIT {count | ALL}] [OFFSET start]
```

## 13、用户自定义函数

udw 支持用户自定义函数，关于用户自定义函数请参考：[官方文档](https://www.postgresql.org/docs/8.3/static/extend.html)
