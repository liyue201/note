### Go日志收集ElasticSearch+Kibana
ELK(ElasticSearch, Logstash, Kibana)是一套经典的日志收集解决方案，这里没有使用Logstash，因为我们的Go服务是用k8s编排，跑在docker中的。
我们使用go的日志库[logrus](https://github.com/sirupsen/logrus)和插件[elogrus.v2](https://github.com/liyue201/elogrus.v2)将日志直接输出到ElasticSearch。

### 主机
* host: 172.13.31.163

### 安装ElasticSearch
安装ElasticSearch，暴露端口9200，数据挂在目录/var/esdata
```
docker run -d  --name elasticsearch -v /var/esdata:/usr/share/elasticsearch/data -p 9200:9200  elasticsearch
```
在浏览器中打开 [http://host:9200/](http://host:9200/)，看到如下json数据，说明安装成功了。

```
{
  "name" : "oJM7rnu",
  "cluster_name" : "elasticsearch",
  "cluster_uuid" : "qoorO-i1Q7Sl5K4Y573iHA",
  "version" : {
    "number" : "5.5.1",
    "build_hash" : "19c13d0",
    "build_date" : "2017-07-18T20:44:24.823Z",
    "build_snapshot" : false,
    "lucene_version" : "6.6.0"
  },
  "tagline" : "You Know, for Search"
}
```

### 安装Kibana

安装Kibana，暴露端口5601，配置ElasticSearch的地址。
```
docker run -d --name kibana -e ELASTICSEARCH_URL=http://172.13.31.163:9200 -p 5601:5601 kibana
```

打开Kibana的页面 [http://host:5601](http://host:5601])。

### 编写测试代码

```
package main

import (
	"github.com/sirupsen/logrus"
	"github.com/liyue201/elogrus.v2"
	"github.com/liyue201/elastic.v5"
)

func main() {
	log := logrus.New()
	client, err := elastic.NewClient(elastic.SetURL("http://172.13.31.163:9200")) //ElasticSearch的地址
	if err != nil {
		log.Panic(err)
	}	
	hook, err := elogrus.NewElasticHook(client, "localhost", logrus.DebugLevel, "mylog") //创建一个mylog的index 
	if err != nil {
		log.Panic(err)
	}	
	log.Hooks.Add(hook)

	log.WithFields(logrus.Fields{
		"name": "joe",
		"age":  42,
	}).Error("Hello world!")
}
```
可以看到代码中创建了一个叫mylog的index。 然后回到Kibana的dashboard中，创建一个名为mylog的index pattern，然后就可以在discover里面搜索日志了。

