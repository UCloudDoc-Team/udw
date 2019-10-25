

# 快速上手

## 一、创建数据仓库

1.选择UDW标签可以跳转到UDW操作界面（如果没有这个标签，请联系客服申请开通），点击欢迎页的“开始探索”，然后点击“创建数据仓库”。

![image](/images/udw2.png)

![image](/images/udw3.png)

2.选择计算节点机型、计算节点数量以及付费方式。

![image](/images/udw4.png)

其中可选的机型配置有：

| 机型       | 名称        | 配置                   |
|------------|-------------|------------------------|
| 存储密集型 | ds1.large   | 4核 24G 2000G(SATA)    |
| 存储密集型 | ds1.6xlarge | 24核 144G 12000G(SATA) |
| 计算密集型 | dc1.large   | 2核 12G 300G(SSD)      |
| 计算密集型 | dc1.8xlarge | 28核 168G 3800G(SSD)   |

选择数据仓库类型：Greenplum 是 EMC 开源的数据仓库产品、Udpg 是基于 PostgreSQL 开发的大规模并行、完全托管的 PB 级数据仓库服务。

选择节点个数：UDW 是分布式架构、所有节点数据都是双机热备,实际可用总容量略小于节点个数\*节点磁盘大小/2，请根据实际数据大小选择合适的节点。

3.设置数据仓库信息

必选项有数据仓库名称、DB管理员用户名、管理员密码。可选项有默认DB,默认DB的名称为dev，你可以选择除了“test”、“postgres”、“template”、“template0”、“template1” 、“default”之外的其他名称。

DB管理员用户名不能为“postgres”。端口固定为 5432，暂不提供修改。

![image](/images/udw5.png)

4.确认支付

![image](/images/udw6.png)

5.等待部署中 数据仓库规模不同，所需要的部署时间会有所差异。

![image](/images/udw7.png)

![image](/images/udw8.png)

## 二、连接数据仓库

![image](/images/udw9.png)

如上图所示客户端访问管理，提供了客户端下载和数据加载工具和文档的下载。

### JDBC连接

Linux操作系统

```
yum install postgresql-jdbc.noarch –y
```

Windows 环境下 JDBC 驱动，将 jar 添加到工程的 BUILD PATH。

1.  示例程序1，java连接UDW，执行建表，插入操作

**PostgreSQLJDBC1.java**

<file java PostgreSQLJDBC1.java>
import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.Statement;
public class PostgreSQLJDBC1 {
    public static void main(String args[]) {
        Connection c = null;
        Statement stmt = null;
        try {
            Class.forName("org.postgresql.Driver");
            c = DriverManager.getConnection("jdbc:postgresql://hostIP:port/dbname",”UserName”,”Password”);
            stmt = c.createStatement();
            String sql = "CREATE TABLE COMPANY " + "(ID INT PRIMARY KEY NOT NULL," + "NAME TEXT NOT NULL," + "AGE INT NOT NULL," + "ADDRESS CHAR(50)," + "SALARY REAL)";
        c.setAutoCommit(false);
        System.out.println("Opened database successfully");
        stmt.executeUpdate(sql);
        sql = "INSERT INTO COMPANY (ID,NAME,AGE,ADDRESS,SALARY) " + "VALUES (1, 'Allen', 25 , 'Texas', 15000.00 );"; 
        stmt.executeUpdate(sql); 
        stmt.close();
        c.commit();
        c.close();
        }
        catch (Exception e) {
            e.printStackTrace();
            System.err.println(e.getClass().getName()+": "+e.getMessage());
            System.exit(0);
        }
        System.out.println("Opened database successfully");
    }
}
</file>

1.  示例程序二：java连接UDW，执行查询操作

<file java PostgreSQLJDBC2.java>
import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.ResultSet;
import java.sql.Statement;
public class PostgreSQLJDBC2 {
     public static void main(String[] args) {
         Connection c = null; 
         Statement stmt = null;
         try{
             Class.forName("org.postgresql.Driver");
             c = DriverManager.getConnection("jdbc:postgresql://hostIP:port/dbname",”UserName”,”Password”);
             stmt = c.createStatement();
             String sql = null;
             System.out.println("Opened database successfully");
             sql = "SELECT * FROM COMPANY;";
             ResultSet res=stmt.executeQuery(sql);
             while(res.next()) {
                 System.out.println(res.getInt(1));
                 System.out.println(res.getString(2));
                 System.out.println(res.getInt(3));
                 System.out.println(res.getString(4));
                 System.out.println(res.getDouble(5));
             }
             stmt.close();
             c.close();
         }
         catch(Exception e) {
             System.err.println( e.getClass().getName()+": "+ e.getMessage() );
             System.exit(0);
         }
     }
}
</file>

### ODBC方式连接

Linux操作系统：CentOS 6.5 64位

1.  安装 postgresql odbc驱动

        yum install postgresql-odbc.x86_64 -y

1.  编辑odbcinst.ini文件，配置odbc驱动

```
vim  /etc/odbcinst.ini

Description    = ODBC for PostgreSQL
Driver         = /usr/lib/psqlodbc.so
Setup          = /usr/lib/libodbcpsqlS.so
Driver64       = /usr/lib64/psqlodbc.so
Setup64        = /usr/lib64/libodbcpsqlS.so
FileUsage      = 1
```

1.  测试ODBC驱动是否安装成功

```
# odbcinst -q -d
[PostgreSQL]
```

如果出现以上输出，代表在这台机器上已成功安装了PostgreSQL的ODBC驱动。

1.  编辑/etc/odbc.int文件配置ODBC连接

```
[testdb]Description  = PostgreSQL connection to TestDB
Driver               = PostgreSQL
Database             = Database
Servername           = MasterNodeIP
UserName             = UserName
Password             = Password
Port                 = Port
Protocol             = 8.3
ReadOnly             = No
RowVersioning        = NoShow
SystemTables         = No
ConnSettings         =
```

1.  测试连接

```
isql testdb
```

![image](/images/udw10.png)

> 如出现以上内容，则表示psqlodbc配置成功。

=== 其他方式 ===

1.udw客户端的方式访问

1.1 udw（greenplum）客户端方式访问（以Centos为例）

如果你选择的数据仓库类型是greenplum、可以采用下面的方式访问

1）下载greenplum客户端解压

```
wget http://udwclient.cn-bj.ufileos.com/greenplum-client.tar.gz

tar -zxvf greenplum-client.tar.gz
```

2）配置udw客户端

进入greenplum-client安装目录，编辑 greenplum\_client\_path.sh 修改UDW\_HOME:export
UDW\_HOME= client安装目录（如/root/greenplum-client）

3） 使配置生效

在\~/.bashrc中添加如下配置

source /data/greenplum-client/greenplum\_client\_path.sh

source \~/.bashrc

备注：/data/greenplum-client是greenplum-client的安装路径

4） 连接数据库

psql -h hostIP（或域名） –U username -d database -p port –W

1.2 udw（udpg）客户端方式访问（以Centos为例）

如果你选择的数据仓库类型是udpg、可以采用下面的方式访问

1）下载udw客户端

```
wget http://udwclient.ufile.ucloud.cn/udw-client.tar
```

解压: tar xvf udw-client.tar

2）配置udw客户端

进入udw-client安装目录，编辑 udw\_client\_path.sh，修改UDW\_CLIENT:export
UDW\_CLIENT= client安装目录（如/root/udw-client）

3）使配置生效在\~/.bashrc中添加如下配置

```
source /data/udw-client/udw_client_path.sh
source ~/.bashrc
```

    备注：/data/udw-client是udw-client的安装路径

4） 连接数据库

psql -h hostIP（或域名） –U username -d database -p port –W

2.python客户端访问

```
$yum install python-psycopg2
```

示例1. 连接UDW testconn.py

``` python
#!/usr/bin/python

import psycopg2
conn = psycopg2.connect(database="dev", user="username", password="password", host="hostIP", port="port")
print "Opened database successfully"
```

执行 python testconn.py

示例2. 创建一个表 createTable.py

``` python
#!/usr/bin/python

import psycopg2
conn = psycopg2.connect(database="dev", user="username", password="password", host="hostIP", port="port")
print "Opened database successfully"
cur = conn.cursor()
cur.execute('''CREATE TABLE COMPANY
    (ID INT PRIMARY KEY     NOT NULL,
    NAME           TEXT    NOT NULL,
    AGE            INT     NOT NULL,
    ADDRESS        CHAR(50),
    SALARY         REAL);''')
 print "Table created successfully"
 conn.commit()
 conn.close()
```

示例3. 插入记录 insert.py

``` python
#!/usr/bin/python

import psycopg2
conn = psycopg2.connect(database="dev", user="username", password="password", host="hostIP", port="port")
print "Opened database successfully"
cur = conn.cursor()
cur.execute("INSERT INTO COMPANY (ID,NAME,AGE,ADDRESS,SALARY) \
  VALUES (1, 'Paul', 32, 'California', 20000.00 )");
cur.execute("INSERT INTO COMPANY (ID,NAME,AGE,ADDRESS,SALARY) \
  VALUES (2, 'Allen', 25, 'Texas', 15000.00 )");
conn.commit()
print "Records created successfully";
conn.close()
```

示例4. 查询 select.py

``` python
#!/usr/bin/python

import psycopg2
conn = psycopg2.connect(database="dev", user="username", password="password", host="hostIP", port="port")
print "Opened database successfully"
cur = conn.cursor()
cur.execute("SELECT id, name, address, salary  from COMPANY")
rows = cur.fetchall()
for row in rows:
    print "ID = ", row[0]
    print "NAME = ", row[1]
    print "ADDRESS = ", row[2]
    print "SALARY = ", row[3], "\n"
print "Operation done successfully";
conn.close()
```

示例5. 更新 update.py

``` python
#!/usr/bin/python

import psycopg2
conn = psycopg2.connect(database="dev", user="username", password="password", host="hostIP", port="port")
print "Opened database successfully"
cur = conn.cursor()
cur.execute("UPDATE COMPANY set SALARY = 25000.00 where ID=1")
conn.commit
print "Total number of rows updated :", cur.rowcount
cur.execute("SELECT id, name, address, salary  from COMPANY")
rows = cur.fetchall()
for row in rows:
   print "ID = ", row[0]
   print "NAME = ", row[1]
   print "ADDRESS = ", row[2]
   print "SALARY = ", row[3], "\n"
 print "Operation done successfully";
 conn.close()
```

示例6. 删除 delete.py

``` python
#!/usr/bin/python

import psycopg2
conn = psycopg2.connect(database="dev", user="username", password="password", host="hostIP", port="port")
print "Opened database successfully"
cur = conn.cursor()
cur.execute("DELETE from COMPANY where ID=2;")
conn.commit
print "Total number of rows deleted :", cur.rowcount
cur.execute("SELECT id, name, address, salary  from COMPANY")
rows = cur.fetchall()
for row in rows:
    print "ID = ", row[0]
    print "NAME = ", row[1]
    print "ADDRESS = ", row[2]
    print "SALARY = ", row[3], "\n"
print "Operation done successfully";
conn.close()
```

3.php客户端

```
yum install php-pgsql
```

示例1. 连接 conn.php

``` php
<?php
$host        = "host=hostIP";
$port        = "port=port";
$dbname      = "dbname=dbname";
$credentials = "user=user password=password";
$db = pg_connect( "$host $port $dbname $credentials"  );
if(!$db){
      echo "Error : Unable to open database\n";
} else {
  echo "Opened database successfully\n";
}
?>
```

示例2. 创建表 create.php

``` php
<?php
$host        = "host=hostIP";
$port        = "port=port";
$dbname      = "dbname=dbname";
$credentials = "user=user password=password";
$db = pg_connect( "$host $port $dbname $credentials"  );
if(!$db){
    echo "Error : Unable to open database\n";
} else {
    echo "Opened database successfully\n";
}
$sql =<<<EOF
  CREATE TABLE COMPANY
  (ID INT PRIMARY KEY NOT NULL,
   NAME TEXT NOT NULL,
   AGE INT NOT NULL,
   ADDRESS CHAR(50),
   SALARY REAL);
EOF;
$ret = pg_query($db, $sql);
if(!$ret)
    { echo pg_last_error($db);
        } else {
          echo "Table created successfullyn";
        }
pg_close($db);
?>
```

示例3. 插入 insert.php

``` php
<?php
$host        = "host=hostIP";
$port        = "port=port";
$dbname      = "dbname=dbname";
$credentials = "user=user password=password";
$db = pg_connect( "$host $port $dbname $credentials"  );
if(!$db){
   echo "Error : Unable to open database\n";
} else {
   echo "Opened database successfully\n";
}
$sql =<<<EOF
   INSERT INTO COMPANY (ID,NAME,AGE,ADDRESS,SALARY)
   VALUES (1, 'Paul', 32, 'California', 20000.00 );
   INSERT INTO COMPANY (ID,NAME,AGE,ADDRESS,SALARY)
   VALUES (2, 'Allen', 25, 'Texas', 15000.00 );
EOF;
$ret = pg_query($db, $sql);
if(!$ret){
   echo pg_last_error($db);
} else {
   echo "Records created successfully\n";
}
pg_close($db);
?>
```

示例4. 查询 select.php

``` php
<?php
$host        = "host=hostIP";
$port        = "port=port";
$dbname      = "dbname=dbname";
$credentials = "user=user password=password";
$db = pg_connect( "$host $port $dbname $credentials"  );
if(!$db){
   echo "Error : Unable to open database\n";
} else {
   echo "Opened database successfully\n";
}
$sql =<<<EOF
   SELECT * from COMPANY;
EOF;
$ret = pg_query($db, $sql);
if(!$ret){
   echo pg_last_error($db);
   exit;
}
while($row = pg_fetch_row($ret)){
   echo "ID = ". $row[0] . "\n";
   echo "NAME = ". $row[1] ."\n";
   echo "ADDRESS = ". $row[2] ."\n";
   echo "SALARY =  ".$row[4] ."\n\n";
}
echo "Operation done successfully\n";
pg_close($db);
?>
```

示例5. 更新 update.php

``` php
<?php
$host        = "host=hostIP";
$port        = "port=port";
$dbname      = "dbname=dbname";
$credentials = "user=user password=password";
$db = pg_connect( "$host $port $dbname $credentials"  );
if(!$db){
   echo "Error : Unable to open database\n";
} else {
   echo "Opened database successfully\n";
}
$sql =<<<EOF
   SELECT * from COMPANY;
EOF;
$ret = pg_query($db, $sql);
if(!$ret){
   echo pg_last_error($db);
   exit;
}
while($row = pg_fetch_row($ret)){
   echo "ID = ". $row[0] . "\n";
   echo "NAME = ". $row[1] ."\n";
   echo "ADDRESS = ". $row[2] ."\n";
   echo "SALARY =  ".$row[4] ."\n\n";
}
echo "Operation done successfully\n";
pg_close($db);
?>
```

示例6. 删除 delete.php

``` php
<?php
$host        = "host=hostIP";
$port        = "port=port";
$dbname      = "dbname=dbname";
$credentials = "user=user password=password";
$db = pg_connect( "$host $port $dbname $credentials"  );
if(!$db){
   echo "Error : Unable to open database\n";
} else {
   echo "Opened database successfully\n";
}
$sql =<<<EOF
   DELETE from COMPANY where ID=2;
EOF;
$ret = pg_query($db, $sql);
if(!$ret){
   echo pg_last_error($db);
   exit;
} else {
   echo "Record deleted successfully\n";
}
$sql =<<<EOF
   SELECT * from COMPANY;
EOF;
$ret = pg_query($db, $sql);
if(!$ret){
   echo pg_last_error($db);
   exit;
}
while($row = pg_fetch_row($ret)){
   echo "ID = ". $row[0] . "\n";
   echo "NAME = ". $row[1] ."\n";
   echo "ADDRESS = ". $row[2] ."\n";
   echo "SALARY =  ".$row[4] ."\n\n";
}
echo "Operation done successfully\n";
pg_close($db);
?>
```

4.SQL Workbench/J 访问 udw

除了以上几种方式，UDW还可以使用SQL Workbench/J来进行访问，详情可见：[SQL Workbench/J 访问
udw](https://static.ucloud.cn/7d32490688f9ddfca7b230c85158785b.pdf)
