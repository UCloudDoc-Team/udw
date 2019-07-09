{{indexmenu_n>80}}

# 数据导入

udw提供了丰富的数据导入方式和工具：

1.  用postgresql的insert和copy方式导入数据到udw
2.  用外部表的方式，把文件并行的导入到udw
3.  创建hdfs的外部表，把hdfs中的数据到udw
4.  通过sqoop把hdfs中的数据导入到udw
5.  用mysql2udw把mysql中的数据导入到udw
6.  创建ufile的外部表、把ufile中数据导入到udw

在导入大量的数据的时候我们建议不要使用insert一条条的导入数据、强烈建议使用copy、udwfile导入数据。

### insert加载数据

-----

我们可以通过insert插入数据到udw，语法如下所示：

INSERT INTO 表名 \[ ( 字段 \[, ...\] ) \] { DEFAULT VALUES | VALUES ( { 表达式
| DEFAULT } \[, ...\] ) | 子查询 }

每次插入一条的效率会比较低、我们建议一次插入多条（2000-5000条）数据。如果要加载的数据量比较大的话、强烈建议使用copy方式加载或者我们下面介绍的几种方式加载。如果您的数据已经在udw中，也可以通过insert
into table1 select \* from table2这种方式加载数据。

### copy加载数据

我们可以用copy快速加载文件数据到udw。具体语法如下：

    cat /data/test.dat | psql -h hostIP -U UserName  -d DB -c "COPY employee from STDIN with CSV DELIMITER '|';"

hostIP：udw访问id

UserName ：访问数据的用户名

DB：数据库名称

employee：表名

### 外部表并行加载数据

外部表并行加载数据是利用http协议实现的一个文件服务器，用于创建udw的外部文件表。使用外部表并行加载数据可以让udw的每个子节点并行的加载数据、大大的加快数据导入udw的速度。在加载数据的时候我们可以先创建一个外部表，然后通过INSERT
INTO \<table\> SELECT \* FROM \<external\_table\>。这样就可以并行的加载文件中的数据。

使用方法请参考我们的文档：[外部表并行加载数据到udw](http://udwclient.ufile.ucloud.com.cn/UDW%E5%A4%96%E9%83%A8%E8%A1%A8.pdf)

### 从hdfs加载数据

为了方便udw和hdfs之间的数据导入和导出，我们提供个两种方案：

1.  用sqoop实现hdfs和udw直接的数据导入导出，使用方法请参考：[hdfs和hive中数据导入导出到udw](http://udwclient.ufile.ucloud.cn/hdfs和hive中数据导入导出到udw.pdf)
    
2.  创建hdfs外部表，使用方法请参考：[创建hdfs外部表](http://udwclient.ufile.ucloud.com.cn/HDFS%E5%A4%96%E9%83%A8%E8%A1%A8.pdf)

### 从mysql中导入数据

为了方便mysql数据导入到udw，我们提供了mysql导入数据到udw的工具mysql2udw，使用方法请参考：[mysql数据导入到udw](http://udwclient.ufile.ucloud.cn/mysql数据导入到udw.pdf)

### 从oracle中导入数据

为了方便oracle数据导入udw，我们提供了oracle导入数据到udw的工具ora2udw,使用方法请参考：
[oracle数据导入到udw](http://ora2udw.ufile.ucloud.com.cn/从Oracle导入数据到UDW.pdf)

### 从ufile加载数据

为了方便ufile数据导入到udw，我们提供了ufile外部表，导入数据到udw，使用方法请参考：[ufile数据导入到udw](http://udwclient.cn-bj.ufileos.com/ufile%E5%AF%BC%E5%85%A5%E5%AF%BC%E5%87%BA%E6%95%B0%E6%8D%AE%E5%88%B0udw.pdf)
