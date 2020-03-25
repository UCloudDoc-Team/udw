# PXF 扩展

在 5.17 及以上版本的 Udw 集群中默认安装了 PXF 扩展服务，Udw 集群可以通过 PXF 服务访问 HDFS, Hive, Hbase 等外部数据，具体使用可以查询对应版本的 [GreenPlum PXF 官方文档](https://gpdb.docs.pivotal.io/5170/pxf/overview_pxf.html)。

使用 PXF 服务访问外部数据时，需要进行一些有关外部数据的配置，我们在控制台提供了配置上传的功能。如果需要访问 Hadoop 相关的外部数据，必须上传对应 Hadoop 集群的 `core-site.xml`, `hdfs-site.xml`, `mapred-site.xml`, `yarn-site.xml` 配置文件，如果还需要额外访问 Hive 或者 Hbase 数据，需要上传 `hive-site.xml` 或者 `hbase-site.xml` 配置文件。

因为配置文件中一般以域名/主机名表示各节点的访问地址，所以还需要额外上传包含 Hadoop 集群各节点的域名/主机名与 IP 对应关系的 `hosts` 文件，我们会将这个文件中的内容添加到 Udw 集群的 `hosts` 文件当中。(请尽量确保上传的 `hosts` 文件只包含集群各节点的 IP 信息，以免造在更新 Udw `hosts` 文件后造成错误)

上传配置之后，需要重启 PXF 服务使配置生效，控制台上提供了 PXF 服务的 停止/开启/重启 的操作功能。
