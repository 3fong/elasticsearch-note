### 常用语法 ###

#### 语法规则 ####


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

实际使用中将PROTOCOL,HOST,PORT省略

#### 文档操作 ####

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
	}'

新版中不推荐指定类型名,实际使用_doc来指明类型.即: 

	POST /megacorp/_doc/2
	{
    	"first_name" : "John2",
    	"last_name" :  "Smith2",
		"age" :        28,
		"about" :      "I love to go rock climbing2",
		"interests": [ "sports2", "music2" ]
	}


- 查询

	批量查询
	
		GET /megacorp/_search
	
	条件查询
	
		方式1: GET /megacorp/_search?q=last_name:Smith

		方式2: 查询last_name=Smith 且 过滤 age <30的数据 
		GET /megacorp/_search
		{
		    "query" : {
		        "bool": {
		            "must": {
		                "match" : {
		                    "last_name" : "Smith" 
		                }
		            },
		            "filter": {
		                "range" : {
		                    "age" : { "lt" : 30 } 
		                }
		            }
		        }
		    }
		}
	

	全文检索:返回所有相关联的数据,类似模糊匹配,但是会返回关联度得分,关联度越高,得分越高

		GET /megacorp/_search
		{
		    "query" : {
		        "match" : {
		            "about" : "rock music"
		        }
		    },
		    "highlight": {
		        "fields" : {
		            "about" : {}
		        }
		    }
		}
	
		highlight:在结果的指定字段中高亮查询条件
	
		匹配短语:match_phrase

	分析(aggregations聚合操作)

		GET /megacorp/_search
		{
		  "aggs": {
		    "all_interests": {
		      "terms": { "field": "interests" }
		    }
		  }
		}

	由于text类型字段默认可查,但是当进行聚合,排序或执行脚本处理时不可查.没有开启聚合优化,造成直接使用聚合操作会失败,开启方式:

		PUT /megacorp/_mapping
		{
		  "properties": {
		    "interests": { 
		      "type":     "text",
		      "fielddata": true
		    }
		  }
		}

[聚合失败原因](https://www.elastic.co/guide/en/elasticsearch/reference/current/text.html#fielddata-mapping-param)	
	
删除索引

DELETE megacorp
