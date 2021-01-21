### Elasticsearch安装和基础概念 ###

#### 简述 ####
Elasticsearch组件安装中 是一个实时的分布式搜索分析引擎,它被用作全文检索、结构化搜索、分析以及这三个功能的组合.实际使用中它一般都是通过集群方式对外服务,所以生产中使用时要注意它的实际应用场景.这里为了学习的目的,采用了Elasticsearch+Kibana的集成方式,通过kibana的可视化开发工具,快速执行elasticsearch语法.

#### 安装Elasticsearch ####

- 选择安装方式,这里使用tar.gz

	wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-7.10.1-linux-x86_64.tar.gz
	wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-7.10.1-linux-x86_64.tar.gz.sha512
	shasum -a 512 -c elasticsearch-7.10.1-linux-x86_64.tar.gz.sha512 
	tar -xzf elasticsearch-7.10.1-linux-x86_64.tar.gz
	cd elasticsearch-7.10.1/ 

- 启动

	./bin/elasticsearch

#### 安装Kibana ####

在安装包括kibana在内的Elasticsearch组件时,要注意保持它们的安装版本与你的Elasticsearch版本的统一,不然可能造成一些命令或者功能适配出现问题.

- 选择安装方式,这里使用tar.gz

    wget https://artifacts.elastic.co/downloads/kibana/kibana-7.10.1-linux-x86_64.tar.gz
    sha1sum kibana-7.10.1-linux-x86_64.tar.gz 
    tar -xzf kibana-7.10.1-linux-x86_64.tar.gz
    cd kibana/ 

- 修改配置文件 config/kibana.yml

	server.port: 5601
	server.host: 172.31.248.173  

这里只设置了端口和主机,主机默认是回环地址,外部不可以访问

- 启动

./bin/kibana

启动后它会通过默认的9200,9300服务端口,自动集成该主机上的elasticsearch服务.

#### 访问kibana ####

- 访问域名 http://YOURDOMAIN.com:5601

进入home主页后->Management -> Dev Tools通过rest api来实际访问elasticsearch

![kibana界面](https://img2020.cnblogs.com/blog/1096086/202101/1096086-20210119230855896-2141746786.png)

#### Elasticsearch几个核心概念 ####

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

创建命令:

	curl -X PUT "localhost:9200/megacorp/employee/1?pretty" -H 'Content-Type: application/json' -d'
	{
	    "first_name" : "John",
	    "last_name" :  "Smith",
	    "age" :        25,
	    "about" :      "I love to go rock climbing",
	    "interests": [ "sports", "music" ]
	}
	'

这里的示例版本是elasticsearch2.X,新版本中推荐创建文档的type指定为_doc,即PUT /megacorp/_doc/1的方式,具体原因可能是通过约定命名的方式来加速检索.

#### 参考资料 ####
[安装Elasticsearch](https://www.elastic.co/guide/en/elasticsearch/reference/7.10/targz.html)
[Elasticsearch重要配置](https://www.elastic.co/guide/en/elasticsearch/reference/7.10/important-settings.html)
[安装kibana](https://www.elastic.co/guide/cn/kibana/current/install.html)
[索引文档](https://www.elastic.co/guide/cn/elasticsearch/guide/current/_indexing_employee_documents.html)
