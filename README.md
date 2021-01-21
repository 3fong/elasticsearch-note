### Elasticsearch study note ###
Elasticsearch 是一个实时的分布式搜索分析引擎,它被用作全文检索、结构化搜索、分析以及这三个功能的组合

	一个分布式的实时文档存储，每个字段 可以被索引与搜索
	一个分布式实时分析搜索引擎
	能胜任上百个服务节点的扩展，并支持 PB 级别的结构化或者非结构化数据

不适用场景:
不需要内容的索引
不需要集群的场景

核心概念:

- node 节点: 集群中的一个服务器,用于存储数据,参与集群的索引和搜索.
- index 
- - 索引(名词):文档分类.类似于关系数据库中的DataBase.
- - 索引(动词):存储一个文档到一个索引中以便于检索.类似于insert关键字.
- - 倒排索引:加速查询列结构.如关系型数据库通过索引如B树(B-tree)索引到指定列,加速检索.默认一个文档每个属性都是被索引(存在倒排索引)的,否则该属性无法被搜索到.
- type 类型:在一个索引中,你可以定义一种或多种类型.一个类型是你索引的一个逻辑分类/分区,其语义由你确定.通常将具有共同字段的文档定义为一个类型.类似于table;
- document 文档:可以被索引的基础信息单元.类似于Record.
- index,type,document关系:
	一个index有多个type,一个type有多个document,一个document有多个属性;

创建员工文档:

	每个员工索引(创建)一个文档，文档包含该员工的所有信息。
	每个文档都将是 employee 类型 。
	该类型位于 索引 megacorp 内。
	该索引保存在我们的 Elasticsearch 集群中。


更轻量的索引 rust语言 见锋亮博客

	
版本环境要求
https://www.elastic.co/cn/support/matrix#matrix_jvm


adduser es

passwd es

chown -R es:es elasticsearch-6.3.2/

chmod 770 elasticsearch-6.3.2/

服务启动配置  https://www.cnblogs.com/michael-xiang/p/10810339.html

配置生效优先级

If you configure the same setting using multiple methods, Elasticsearch applies the settings in following order of precedence:

Transient setting
Persistent setting 
elasticsearch.yml setting
Default setting value

开发模式和生产模式
network.host

By default, Elasticsearch assumes that you are working in development mode. If any of the above settings are not configured correctly, a warning will be written to the log file, but you will be able to start and run your Elasticsearch node.

As soon as you configure a network setting like network.host, Elasticsearch assumes that you are moving to production and will upgrade the above warnings to exceptions. These exceptions will prevent your Elasticsearch node from starting. This is an important safety measure to ensure that you will not lose data because of a malconfigured server.

If Elasticsearch is in development mode, any bootstrap checks that fail appear as warnings in the Elasticsearch log. If Elasticsearch is in production mode, any bootstrap checks that fail will cause Elasticsearch to refuse to start.


临时文件配置


Temporary directory settingsedit
By default, Elasticsearch uses a private temporary directory that the startup script creates immediately below the system temporary directory.

On some Linux distributions, a system utility will clean files and directories from /tmp if they have not been recently accessed. This behavior can lead to the private temporary directory being removed while Elasticsearch is running if features that require the temporary directory are not used for a long time. Removing the private temporary directory causes problems if a feature that requires this directory is subsequently used.

If you install Elasticsearch using the .deb or .rpm packages and run it under systemd, the private temporary directory that Elasticsearch uses is excluded from periodic cleanup.

If you intend to run the .tar.gz distribution on Linux or MacOS for an extended period, consider creating a dedicated temporary directory for Elasticsearch that is not under a path that will have old files and directories cleaned from it. This directory should have permissions set so that only the user that Elasticsearch runs as can access it. Then, set the $ES_TMPDIR environment variable to point to this directory before starting Elasticsearch.



Virtual memoryedit
Elasticsearch uses a mmapfs directory by default to store its indices. The default operating system limits on mmap counts is likely to be too low, which may result in out of memory exceptions.

On Linux, you can increase the limits by running the following command as root:

    sysctl -w vm.max_map_count=262144

To set this value permanently, update the vm.max_map_count setting in /etc/sysctl.conf. To verify after rebooting, run sysctl vm.max_map_count.

The RPM and Debian packages will configure this setting automatically. No further configuration is required.

Maximum map count checkedit
Continuing from the previous point, to use mmap effectively, Elasticsearch also requires the ability to create many memory-mapped areas. The maximum map count check checks that the kernel allows a process to have at least 262,144 memory-mapped areas and is enforced on Linux only. To pass the maximum map count check, you must configure vm.max_map_count via sysctl to be at least 262144.

Alternatively, the maximum map count check is only needed if you are using mmapfs or hybridfs as the store type for your indices. If you do not allow the use of mmap then this bootstrap check will not be enforced.


Discovery configuration checkedit
By default, when Elasticsearch first starts up it will try and discover other nodes running on the same host. If no elected master can be discovered within a few seconds then Elasticsearch will form a cluster that includes any other nodes that were discovered. It is useful to be able to form this cluster without any extra configuration in development mode, but this is unsuitable for production because it’s possible to form multiple clusters and lose data as a result.

This bootstrap check ensures that discovery is not running with the default configuration. It can be satisfied by setting at least one of the following properties:

discovery.seed_hosts
discovery.seed_providers
cluster.initial_master_nodes


path.data
Every data and master-eligible node requires access to a data directory where shards and index and cluster metadata will be stored. The path.data defaults to $ES_HOME/data but can be configured in the elasticsearch.yml config file an absolute path or a path relative to $ES_HOME as follows:

path.data:  /var/elasticsearch/data
Like all node settings, it can also be specified on the command line as:

./bin/elasticsearch -Epath.data=/var/elasticsearch/data


与ElasticSearch交互:

rest

	curl -X<VERB> '<PROTOCOL>://<HOST>:<PORT>/<PATH>?<QUERY_STRING>' -d '<BODY>'

被 < > 标记的部件：

VERB

适当的 HTTP 方法 或 谓词 : GET、 POST、 PUT、 HEAD 或者 DELETE。

PROTOCOL

http 或者 https（如果你在 Elasticsearch 前面有一个 https 代理）

HOST

Elasticsearch 集群中任意节点的主机名，或者用 localhost 代表本地机器上的节点。

PORT

运行 Elasticsearch HTTP 服务的端口号，默认是 9200 。

PATH

API 的终端路径（例如 _count 将返回集群中文档数量）。Path 可能包含多个组件，例如：_cluster/stats 和 _nodes/stats/jvm 。

QUERY_STRING

任意可选的查询字符串参数 (例如 ?pretty 将格式化地输出 JSON 返回值，使其更容易阅读)

BODY

一个 JSON 格式的请求体 (如果请求需要的话)

