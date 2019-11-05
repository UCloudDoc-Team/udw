# 访问UDW数据仓库

## 1 客户端工具访问UDW

udw支持按照postgresql的客户端来访问udw，支持udw客户端访问，还可以支持jdbc、odbc、php、python、psql等方式来访问udw。另外，也可以通过图形化的SQL
Workbench/J、Navicat等工具来访问udw。

### 1.1 psql客户端方式访问

下载psql客户端

    yum install postgresql.x86_64 (64位系统)

    psql -h hostIP（或域名） –U username -d database -p port –W

> hostIP：udw master节点的ip或者域名
> username: 数据库用户名
> database：数据库名称

### 1.2 udw客户端方式访问

如果你选择是数据仓库类型是greenplum、请用greenplum客户端、如果你选择的数据仓库类型请用udpg客户端。

1.1 udw（greenplum）客户端方式访问（以Centos为例）

1）下载greenplum客户端解压

    wget <http://udw.cn-bj.ufileos.com/greenplum-client.tar.gz>

    tar -zxvf greenplum-client.tar.gz

2）配置udw客户端

进入greenplum-client安装目录，编辑 `greenplum_client_path.sh`

修改`UDW_HOME`

    export UDW_HOME= client安装目录（如/root/greenplum-client）

3） 使配置生效

在`~/.bashrc`中添加如下配置

    source /data/greenplum-client/greenplum_client_path.sh

执行

    source ~/.bashrc

> 备注：/data/greenplum-client是greenplum-client的安装路径

4） 连接数据库

    psql -h hostIP（或域名） –U username -d database -p port –W

1.2 udw（udpg）客户端方式访问（以Centos为例）

1）下载udw客户端

    wget http://udw.cn-bj.ufileos.com/udw-client.tar
    tar xvf udw-client.tar

2）配置udw客户端

进入udw-client安装目录，编辑 `udw_client_path.sh`

修改`UDW_CLIENT`:

    export UDW_CLIENT=client安装目录（如/root/udw-client）

3）使配置生效在`~/.bashrc`中添加如下配置

    source /data/udw-client/udw_client_path.sh

    source ~/.bashrc

> 备注：/data/udw-client是udw-client的安装路径

4） 连接数据库

    psql -h hostIP（或域名） –U username -d database -p port –W

### 1.3 JDBC方式访问

Linux操作系统

```
yum install postgresql-jdbc.noarch –y
```

Windows环境下JDBC驱动，将jar添加到工程的BUILD PATH。

示例程序1，java连接UDW，执行建表，插入操作

**PostgreSQLJDBC1.java**

``` java
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
```

示例程序二：java连接UDW，执行查询操作



``` java
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
```

### 1.4 ODBC方式访问

Linux操作系统：CentOS 6.5 64位

1. 安装 postgresql odbc驱动

        yum install postgresql-odbc.x86_64 -y

2. 编辑`/etc/odbcinst.ini`文件，配置odbc驱动

        Description    = ODBC for PostgreSQL
        Driver         = /usr/lib/psqlodbc.so
        Setup          = /usr/lib/libodbcpsqlS.so
        Driver64       = /usr/lib64/psqlodbc.so
        Setup64        = /usr/lib64/libodbcpsqlS.so
        FileUsage      = 1

3. 测试ODBC驱动是否安装成功

        # odbcinst -q -d
        [PostgreSQL]

    如果出现以上输出，代表在这台机器上已成功安装了PostgreSQL的ODBC驱动。

4. 编辑`/etc/odbc.ini`文件配置ODBC连接

        [testdb] Description  = PostgreSQL connection to TestDB
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

5.  测试连接

        # isql testdb

    ![image](/images/udw10.png)

> 注解：
>
> 如出现以上内容，则表示psqlodbc配置成功。

### 1.5 python客户端访问

    $yum install python-psycopg2

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

### 1.6 php客户端访问

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

### 1.7 node客户端访问

1）安装pg模块

    npm install -g node_gyp

    npm install -g pg

2）连接数据库并访问

示例代码如下：

``` javascript
var pg = require('pg');
var constring = "tcp://username:password@ip:port/database";
var client = new pg.Client(constring);
client.connect();
client.query("create temp table beatle(name varchar(10),height integer)");
client.query("insert into beatle(name,height) values('john',50)");
client.query("insert into beatle(name,height) values($1,$2)",['brown',68]);
var query = client.query("select * from beatle");
query.on('row',function(row){
    console.log(row);
});
query.on('end',function(){
    client.end();
});
```

如果连接成功，输出：

    { name: 'john', height: 50 }

    { name: 'brown', height: 68 }

## 2 图形界面的方式访问UDW

### 2.1 配置UDW外网访问

udw默认是通过内网访问的，为了数据安全性，尽量不要通过外网访问UDW，如果需要图形界面的方式访问UDW，则需要配置udw的外网访问，请参考：

前提：有一台可以访问 udw 的 uhost，并且这台 uhost 上可以访问外网 ip。

例如：uhost 内网 ip 是 10.10.0.9 外网 IP 是 192.168.120.110，udw 的访问ip 是 10.10.10.1，

我们在 uhost 机器上建立 ssh 隧道即可通过 192.168.120.110访问 udw。在 uhost 机器上执行如下命令：

    ssh -C -f -N -g -L 5432:10.10.10.1:5432 root@10.10.0.9

备注：请注意开放外网防火墙端口 5432（也可以把 udw 端口映射到 uhost上其他端口上），网络防火墙配置请参考： <https://docs.ucloud.cn/network/firewall/index.html>

### 2.2 SQL Workbench/J

SQL Workbench/J是一个独立于DBMS，跨平台的SQL查询分析工具。具有通用性好、小巧、免安装等优点，

并且功能强大，查询编辑器支持自动补全，Database Explorer可以查看和编辑各种数据库对象（表、视图、存储过程等）。

详情可见：[SQL Workbench/J 访问 udw](http://udw.cn-bj.ufileos.com/SQL%20Workbench%3AJ%20%E8%AE%BF%E9%97%AE%20udw.pdf)
