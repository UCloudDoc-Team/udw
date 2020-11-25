# 使用 pg_dump 迁移数据

## 安装 greenplum-db-clients

为了获取 `pg_dump` 工具，需要安装 greenplum-db-clients，安装方法可以查看 <https://gpdb.docs.pivotal.io/6-2/client_tool_guides/installing.html>，根据指引下载适用的安装包并进行安装。

以 CentOS 7.x 为例，下载 6.2.1 的安装包 `greenplum-db-clients-6.2.1-rhel7-x86_64.rpm`，执行命令 `yum install greenplum-db-clients-6.2.1-rhel7-x86_64.rpm` 进行安装。安装之后将 `/usr/local/greenplum-db-clients/bin` 加入系统 `PATH` 环境变量。

## 使用 pg_dump 导出数据

pg_dump 完整说明可以查看文档 <https://www.postgresql.org/docs/9.4/app-pgdump.html>，常用参数如下：

* `-d dbname`, `--dbname=dbname`：指定 database
* `-h host`, `--host=host`：指定连接地址/IP
* `-p port`, `--port=port`：指定访问端口
* `-U username`, `--username=username`：指定访问用户名
* `-W`, `--password`：提示输入密码

* `-a`, `--data-only`：只导出数据
* `-s`, `--schema-only`：只导出定义
* `-n schema`, `--schema=schema`：指定 schema，可以使用多次 `-n` 指定多个 schema
* `-t table`, `--table=table`：指定 table，可以使用多次 `-t` 指定多个 table

示例如下：

导出 `public` schema 下所有表定义到 `public_all.sql` 文件：

    pg_dump -d postgres -h 10.9.141.93 -p 5432 -U admin -W \
            -s -t "public.*" -f public_all.sql

导出 `public` schema 下 `src` table 的定义到 `public_src.sql` 文件：

    pg_dump -d postgres -h 10.9.141.93 -p 5432 -U admin -W \
            -s -t "public.src" -f public_src.sql

导出 `public` schema 下所有表数据到 `public_all_data.sql`：

    pg_dump -d postgres -h 10.9.141.93 -p 5432 -U admin -W \
            -a -t "public.*" -f public_all_data.sql

导出 `public` schema 下 `src` table 的数据到 `public_src_data.sql` 文件：

    pg_dump -d postgres -h 10.9.141.93 -p 5432 -U admin -W \
            -a -t "public.src" -f public_src_data.sql

## 使用 psql 重建数据

重新创建 `public` schema 下所有表：

    psql -d postgres -h 10.9.52.209 -p 5432 -U admin -W -f public_all.sql

重新写入 `public` schema 下所有表的数据：

    psql -d postgres -h 10.9.52.209 -p 5432 -U admin -W -f public_all_data.sql

