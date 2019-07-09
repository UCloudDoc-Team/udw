{{indexmenu_n>95}}

UDW可以接入第三方商业智能（BI）工具来快速实现数据的可视化。第三方商业智能（BI）工具使用标准数据库接口连接UDW数据仓库，例如：JDBC和ODBC。

目前经过测试的有：Zeppelin和SuperSet。

# 一、 UDW接入Zeppelin

### Zeppelin简介

Zeppelin是一个开源的Apache的孵化项目. 它是一款基本web的notebook工具，支持交互式数据分析。通过插件的方式接入各种解释器（interpreter），使得用户能够以特定的语言或数据处理后端来完成交互式查询，并快速实现数据可视化。

### 部署Zeppelin

1\) 安装Java

Zeppelin支持的操作系统如下图所示。在安装Zeppelin之前，你需要在部署的服务器上安装Oracle JDK 1.7或以上版本,
并配置好相应的JAVA\_HOME环境变量。

![image](/images/zeppelin_1.png)

以CentOS为例，具体操作过程如下：

a) 下载JDK安装包(jdk-7u79-linux-x64.tar.gz),下载地址为：

<http://www.oracle.com/technetwork/cn/java/javase/downloads/jdk7-downloads-1880260.html>。

创建JDK安装目录，并将安装包解压至该目录：

mkdir /usr/java

tar zxvf jdk-7u79-linux-x64.tar.gz

a) 建立软链接

ln -s /usr/java/jdk1.7.0\_79 /usr/java/java

b) 配置环境变量。在/etc/profile文件结尾添加：

export JAVA\_HOME=/usr/java/java

export JRE\_HOME=${JAVA\_HOME}/jre

export PATH=${JAVA\_HOME}/bin:$PATH

d) 使环境变量生效

source /etc/profile

2\) 获取Zeppelin

下载地址：<http://zeppelin.apache.org/download.html>

选择二进制安装包，这里以zeppelin-0.6.2-bin-all.tgz为例。

3）安装Zeppelin

安装Zeppelin只需如下命令解压二进制安装包即可：

tar zxvf zeppelin-0.6.2-bin-all.tgz

启动Zeppelin:

cd /data/zeppelin-0.6.2-bin-all (Zeppelin的安装目录)

bin/zeppelin-daemon.sh start

第一次启动Zeppelin，输出如下：

![image](/images/zeppelin_2.png)

这说明Zeppelin已经部署成功。

4）验证

Zeppelin默认启动在8080端口，在浏览器中访问Zeppelin主页，访问地址是:
<http://your_host_ip:8080/,你将看到类似如下的页面>。

![image](/images/zeppelin_3.png)

### Zeppelin接入UDW数据仓库

1）为UDW创建一个interpreter。

鼠标点击右上角的“anonymous”，在弹出的 下拉列表中选择“Interpreter”。

![image](/images/zeppelin_4.png)

你将进入如下页面，然后点击右上角的“+Create”按钮。

![image](/images/zeppelin_5.png)

接着，便进入了解释器的新建页面，如下图：

![image](/images/zeppelin_6.png)

输入解释器的名字（可任意），解释器分组选择jdbc。

保留并修改上图中连接数据仓库的配置参数，点选action下面的“x”删除其他无关参数。然后点击“Save”按钮，保存设置。

2）创建笔记

现在，你可以新建笔记来测试该Interpreter了。

鼠标点击“Notebook”下方的“Create new note”创建一个笔记。

![image](/images/zeppelin_7.png)

输入笔记名称（任意），点击“Create Note”，如下图：

![image](/images/zeppelin_8.png)

笔记创建完后，点击“齿轮”图标进行设置，将之前创建的解释器UDW移到最上方，作为默认的Interpreter使用。点击“Save”保存设置。

![image](/images/zeppelin_9.png)

3） 操作数据仓库

在方框内输入SQL语句，点击下图中的“▶”按钮，执行SQL。

![image](/images/zeppelin_10.png)

执行结束，输出如下。

![image](/images/zeppelin_11.png)

### 数据可视化

Zeppelin提供了非常丰富且简单的可视化功能，点击如下图中的可视化选项，拖动fields完成简单的setting设置，即可看到不同种类的可视化图表了。

![image](/images/zeppelin_12.png)

Zeppelin提供的5种可视化视图如下所示：

![image](/images/zeppelin_13.png)

![image](/images/zeppelin_14.png)

![image](/images/zeppelin_15.png)

![image](/images/zeppelin_16.png)

![image](/images/zeppelin_17.png)

# 二、 UDW接入SuperSet

### SuperSet简介

Superset （Caravel） 是由Airbnb（知名在线房屋短租公司）开源的数据分析与可视化平台（曾用名Caravel、Panoramix），该工具主要特点是可自助分析、自定义仪表盘、分析结果可视化（导出）、用户/角色权限控制，还集成了一个SQL编辑器，可以进行SQL编辑查询等。

### 部署SuperSet

以centos
64操作系统为例(其他操作系统可参考：<http://airbnb.io/superset/installation.html>）：

1\) 安装Python3及组件

SuperSet只支持Python2.7和Python3.4以上版本。SuperSet官网建议使用Python3。

安装系统依赖：

yum install gcc gcc-c++ libffi-devel python-devel python-pip
python-wheel openssl-devel libsasl2-devel openldap-devel sqlite-devel
zlib-devel bzip2-devel openssl-devel ncurses-devel postgresql-devel -y

安装Python3:

wget <https://www.python.org/ftp/python/3.5.0/Python-3.5.0.tar.xz>

tar Jxvf Python-3.5.0.tar.xz

cd Python-3.5.0

```
./configure --prefix=/usr/local/python3
```

make && make install

```
echo 'export PATH=$PATH:/usr/local/python3/bin' >> ~/.bashrc
```

```
source ~/.bashrc
```

rm /usr/bin/python

ln -sv /usr/local/python3/bin/python3.5 /usr/bin/python

更新yum配置

编辑/usr/bin/yum

将第一行的\#！/usr/bin/python改为\#\!/usr/bin/python2.6，保存退出。

至此完成了python3的安装。

安装fab

wget <https://pypi.python.org/packages/source/f/fab/fab-1.4.2.tar.gz>

tar zxvf fab-1.4.2.tar.gz

cd fab-1.4.2

python setup.py install

升级pip

```
pip install --upgrade pip
```

备注：如果pip升级过程报版本错误，请执行下面操作 请先mv /usr/bin/pip /usr/bin/pip.bak 然后执行 ln -s
/usr/local/python3/bin/pip /usr/bin/pip

安装psycopg2

pip install psycopg2==2.6.2

下载http://udwclient.cn-bj.ufileos.com/extras.py，替换/usr/local/python3/lib/python3.5/site-packages/psycopg2/下的extras.py文件。

2）安装SuperSet

pip install superset

创建管理用户(后面登录web页面的时候会用到)

```
fabmanager create-admin --app superset
```

初始化数据库

superset db upgrade

创建默认角色和权限

superset init

更新sqlalchemy

pip install sqlalchemy==1.0.16

在启动服务之前，还需要修改如下：

1、下载http://udwclient.cn-bj.ufileos.com/base.py，替换/usr/local/python3/lib/python3.5/site-packages/sqlalchemy/dialects/postgresql目录下的base.py。

2、下载http://udwclient.cn-bj.ufileos.com/default.py，替换/usr/local/python3/lib/python3.5/site-packages/sqlalchemy/engine目录下的default.py。

在8088端口启动web服务器(注意修改相应的防火墙保证8088端口可以被访问)

superset runserver -p 8088

3）验证

SuperSet默认启动在8088端口，在浏览器中访问SuperSet主页，访问地址是:
<http://your_host_ip:8088/,你将看到类似如下的登录页面>。

![image](/images/superset_1.png)

登录之后将看到如下页面：

![image](/images/superset_2.png)

### SuperSet接入UDW数据仓库

1）创建数据源(Databases)

![image](/images/superset_3.png)

输入参数，测试连接

![image](/images/superset_4.png)

这里需要注意Sqlalchemy Uri的写法：

前缀是：postgresql+psycopg2，后缀是：username:password@host:port/database

点击“TEST CONNECTION”，提示测试连接成功，并且在最下方，列出了数据库dev中所有的表。

2）执行sql

SuperSet集成了一个SQL编辑器，点击“SQL
Editor”，选择schema(不选的话是默认的schema，一般是public），选择一个表可以预览该表的数据，如下图所示：

![image](/images/superset_16.png)

选择Database，输入SQL，点击“Run Query”，获取查询结果（注意，此时不要选择schema，否则会报错），如下图所示：

![image](/images/superset_5.png)

### 数据可视化

SuperSet支持十几种可视化图表，用于将查询返回的数据做可视化展示。

以上面的查询为例，点击“Visualize”进入可视化配置页面如下：

![image](/images/superset_6.png)

选择维度，勾选Sum、Min、Max、Count Distinct选项，点击“Visualize”则会生成相应的可视化页面。

![image](/images/superset_7.png)

点击Save as可以将一个定制好的数据探索保存成Slice，多个Slice可以组成一个Dashboard。

![image](/images/superset_8.png)

查看所有的slice:

![image](/images/superset_9.png)

在添加Dashboard页面，指定包含哪些Slice，定制自己的Dashboard。

![image](/images/superset_10.png)

![image](/images/superset_11.png)

其中，每个Slice对应的模块，可以自由拖拽位置和大小，并保存整个Dashboard的布局。

关于superset的更多信息请参考：

<http://airbnb.io/superset/>

<https://github.com/airbnb/superset>
