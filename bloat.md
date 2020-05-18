# 表膨胀

## 表膨胀的原因

udw的存储实现(MVCC-多版本并发控制)来自于Postgres。根据MVCC的原理，没有办法直接更新数据(更新操作(update)是通过先删除(delete)再插入(insert)实现的)，被更新之前的行数据仍然在数据文件中。

## 如何避免表膨胀

### 方法一：vacuum full table

> vacuum full不能回收索引的膨胀空间。
> vacuum full 加载的锁与 DDL 锁类似，是排它锁。建议在没有业务的时候执行，不要堵塞业务。

使用 vacuum full 回收垃圾的建议操作流程：

1. 记录下表的索引
2. 删除索引
3. vacuum full 表
4. 重建索引

示例：

创建测试表：

```
dev=# create table test(id int, name text);
NOTICE:  Table doesn't have 'DISTRIBUTED BY' clause -- Using column named 'id' as the Greenplum Database data distribution key for this table.
HINT:  The 'DISTRIBUTED BY' clause determines the distribution of data. Make sure column(s) chosen are the optimal data distribution key to minimize skew.
CREATE TABLE

dev=# insert into test select generate_series(1,100000000), 'test';
INSERT 0 100000000

dev=# create index idx_test on test(id);
CREATE INDEX

dev=# update test set name='nice';
UPDATE 100000000
```

查看表格数据：

```
dev=# select pg_size_pretty(pg_relation_size('test'));
 pg_size_pretty
----------------
 8401 MB
(1 row)
```

查看索引数据：

```
dev=# select pg_size_pretty(pg_relation_size('idx_test'));
 pg_size_pretty
----------------
 6377 MB
(1 row)
```

先回收表数据（此方法不能回收索引数据）：

```
dev=# vacuum full test;
VACUUM

dev=# select pg_size_pretty(pg_relation_size('test'));
 pg_size_pretty
----------------
 4200 MB
(1 row)

Time: 4.278 ms
dev=# select pg_size_pretty(pg_relation_size('idx_test'));
 pg_size_pretty
----------------
 6377 MB
(1 row)
```

回收索引和表的数据：

```
dev=# drop index idx_test;
DROP INDEX

dev=# vacuum full test;
VACUUM

dev=# create index idx_test on test(id);
CREATE INDEX

dev=# select pg_size_pretty(pg_relation_size('test'));
 pg_size_pretty
----------------
 4200 MB
(1 row)

dev=# select pg_size_pretty(pg_relation_size('idx_test'));
 pg_size_pretty
----------------
 2126 MB
(1 row)
```

### 方法二：通过修改分布键释放空间

修改分布键可以回收索引的膨胀空间。修改分布键加载的锁与 DDL 锁类似，是排它锁。建议在没有业务的时候执行，不要影响业务。

```
alter table test set with (reorganize=true) distributed randomly;
alter table test set with (reorganize=true) distributed by (id);
```


实例：

```
dev=# update test set name='back';
UPDATE 100000000
```

查看数据：

```
dev=# select pg_size_pretty(pg_relation_size('test'));
 pg_size_pretty
----------------
 8401 MB
(1 row)

dev=# select pg_size_pretty(pg_relation_size('idx_test'));
 pg_size_pretty
----------------
 4251 MB
(1 row)
```

查看表格的分布键，如下所示Distributed by: (id)，分布键为id

```
dev=# \d+ test
                  Table "public.test"
 Column |  Type   | Modifiers | Storage  | Description
--------+---------+-----------+----------+-------------
 id     | integer |           | plain    |
 name   | text    |           | extended |
Indexes:
    "idx_test" btree (id)
Has OIDs: no
Distributed by: (id)
```

按照原有的分布键重新分布

```
dev=# alter table test set with (reorganize=true) distributed by (id);
ALTER TABLE
```

查看数据

```
dev=# select pg_size_pretty(pg_relation_size('test'));
 pg_size_pretty
----------------
 4200 MB
(1 row)

dev=# select pg_size_pretty(pg_relation_size('idx_test'));
 pg_size_pretty
----------------
 2126 MB
```

### 方法三：创建新表，导入数据

`CREATE TABLE...AS SELECT` 命令把该表拷贝为一个新表，新建的表将不会出现膨胀现象。然后删除原始表并且重命名拷贝的表。

参考：
* <https://gp-docs-cn.github.io/docs/best_practices/bloat.html>
* <https://docs.ucloud.cn/udw/developer?id=_43-%e9%80%89%e6%8b%a9%e6%95%b0%e6%8d%ae%e5%88%86%e5%b8%83%e7%ad%96%e7%95%a5>
