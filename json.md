# UDW中Json类型

## Json相关操作

| 操作符 | 参数类型 | 作用                                             | 例子                                                        | 执行结果     |
|--------|----------|--------------------------------------------------|-------------------------------------------------------------|--------------|
| \-\>   | int      | 获取JSON数组元素，索引以0为开始                  | select '\[{"a":"foo"},{"b":"bar"},{"c":"baz"}\]'::json-\>2; | {"c":"baz"}  |
| \-\>   | text     | 通过键来获取 JSON 对象的域（field）              | select '{"a": {"b":"foo"}}'::json-\>'a';                    | {"b":"foo"}  |
| \-\>\> | int      | 获取 JSON 数组元素，然后以 text 形式返回它       | select '\[1,2,3\]'::json-\>\>2;                             | 3            |
| \-\>\> | text     | 获取 JSON 对象的域，然后以 text 形式返回它       | select '{"a":1,"b":2}'::json-\>\>'b';                       | 2            |
| \#\>   | text\[\] | 获取指定路径上的 JSON 对象                       | select '{"a": {"b":{"c": "foo"}}}'::json\#\>'{a,b}';        | {"c": "foo"} |
| \#\>\> | text\[\] | 获取指定路径上的 JSON 对象，并以 text 形式返回它 | select '{"a":\[1,2,3\],"b":\[4,5,6\]}'::json\#\>\>'{a,2}';  | 3            |

## Json操作举例

创建json类型的表格，插入数据

``` sql
create table test_json(id int , name json)
with (APPENDONLY=true,ORIENTATION=column,compresslevel=5) DISTRIBUTED BY (id);
INSERT INTO test_json VALUES 
(1,'{ "id": 1, "sub": { "subid": 10,"subsub": {"subsubid": 100}}}'),
(2,'{ "id": 20, "sub": { "subid": 200,"subsub": {"subsubid": 2000}}}'),
(1,'{ "id": 1, "sub": { "subid": "test","subsub": {"subsubid": 100}}}'),
(3,'{ "id": 1,"sub":"test","name":"me","ip":"10.10.10.10" }');
```

json操作类型操作举例：

``` sql

select * from test_json where name->>'id'=1;
 id |                               name                                
----+-------------------------------------------------------------------
  1 | { "id": 1, "sub": { "subid": 10,"subsub": {"subsubid": 100}}}
  1 | { "id": 1, "sub": { "subid": "test","subsub": {"subsubid": 100}}}
  3 | { "id": 1,"sub":"test","name":"me","ip":"10.10.10.10" }


select * from test_json where name->'sub'->>'subid'=10;
 id |                             name                              
----+---------------------------------------------------------------
  1 | { "id": 1, "sub": { "subid": 10,"subsub": {"subsubid": 100}}}
  
select * from test_json where name->>'name'='me';
 id |                          name                           
----+---------------------------------------------------------
  3 | { "id": 1,"sub":"test","name":"me","ip":"10.10.10.10" }


```

## Json相关函数

### Json创建函数

``` json
to_json(anyelement) 
array_to_json(anyarray [, pretty_bool])
row_to_json(record [,pretty_bool])
json_build_array(VARIADIC "any")
json_build_object(VARIADIC "any")
json_object(text[]) 
json_object(keys text[], values text[]) 
```

### Json处理函数

``` json
json_array_length(json)
json_extract_path(from_json json, VARIADIC path_elems text[])
json_extract_path_text(from_json json, VARIADICpath_elems text[])
json_object_keys(json)
json_populate_record( base anyelement, from_json json)
json_typeof(json)
json_to_record(json)
json_to_recordset(json)
```

> json函数的详细操作请参考文档下面的部分。

## Json创建函数

``` json
to_json(anyelement) 
```

以 JSON 格式返回输入的值。 数组和复合数据会被（递归地）转换为数组和对象；
如果有转换函数可以将输入的数据转换为 json 的话，那么使用转换函数；
或者产生一个 JSON 标量（scalar）值。 数字、布尔值和空值（null）之外的其他标量会被表示为文本格式，
并通过正确的引用和转义来保证它是一个合法的 JSON 字符串。如下所示：

![](/images/uhadoop_json1.png)

``` json
array_to_json(anyarray [, pretty_bool]) 
```

以 JSON 数组格式返回输入的数组。 一个UDW多维数组将被转换成一个由多个数组组成的 JSON
数组。如果 pretty\_bool的值为 true ， 那么则在维度-1元素之间添加换行符。 如下所示：

![](/images/uhadoop_json2.png)

``` json
row_to_json(record [,pretty_bool])
```

以 JSON 对象格式返回行。如果pretty\_bool为 true， 将在级别-1元素之间添加换行符。

![](/images/uhadoop_json3.png)

``` json
json_build_array(VARIADIC "any")
```

建立一个可能不同类型的JSON数组，由可变参数列表组成。例如：

![](/images/uhadoop_json4.png)

``` json
json_build_object(VARIADIC "any")
```

建立一个JSON对象的可变参数列表。根据习惯，该参数列表由交替的键和值组成。例如：

![](/images/uhadoop_json5.png)

``` json
json_object(text[]) 
```

输入的文本数组构建一个 JSON 对象。 输入的数组要么就是由偶数个成员组成的一维数组， 数组中的每两个成员组成一个键值对；
要么就是一个二维数组，并且每个内部数组都正好包含两个元素， 这两个元素组成一个键值对。例如：

![](/images/uhadoop_json6.png)

``` json
json_object(keys text[], values text[]) 
```

这个格式的 json\_object 函数接受两个数组作为输入， 第一个数组的元素会被用作键值对的键， 而第二个数组的元素则会被用作键值对的值。

![](/images/uhadoop_json7.png)

## Json处理函数

``` json
json_array_length(json)
```

返回最外层的 JSON 数组的元素数量。例如：

![](/images/uhadoop_json8.png)

``` json
json_extract_path(from_json json, VARIADIC path_elems text[])
```

返回 path\_elems 所指向的 JSON 值。 等同于 \#\> 操作符。例如：

![](/images/uhadoop_json9.png)

``` json
json_extract_path_text(from_json json, VARIADICpath_elems text[])
```

以 text 格式， 返回 path\_elems 所指向的 JSON 值。 效果等同于 \#\>\> 操作符。例如：

![](/images/uhadoop_json10.png)

``` json
json_object_keys(json)
```

返回最外层的 JSON 对象所包含的键。 例如：

![](/images/uhadoop_json11.png)

``` json
json_populate_record( base anyelement, from_json json)
```

将 from\_json 中的对象展开到一个行里面， 这个行的各个列与 base 中定义的 record 类型一致。例如：

![](/images/uhadoop_json12.png)

``` json
json_typeof(json)
```

以字符串形式返回最外层 JSON 值的类型。可能出现的类型有 object 、 array、string、number 、boolean 和
null 。例如：

![](/images/uhadoop_json13.png)

``` json
json_to_record(json)
```

根据一个 JSON 对象来构建一个任意的 record 。 和所有返回 record 的函数一样， 调用者必须通过 as 语句来明确地定义
record 的结构。

![](/images/uhadoop_json14.png)

``` json
json_to_recordset(json)
```

根据一个由 JSON 对象组成的数组， 构建一个任意的 record 集合。 和所有返回 record 的函数一样， 调用者必须通过 as
语句来明确地定义 record 的结构。例如： ![](/analysis/udw/uhadoop_json15.png)
