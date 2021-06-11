---
layout: post
title: "es 入门"
date: 2021-05-19 15:50
comments: true
reward: false
top: false
brief: ""
tags: 
	- elasticsearch kibana
key: "elasticsearch kibana"
---

# es 安装
wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-7.12.1-linux-x86_64.tar.gz
wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-7.12.1-linux-x86_64.tar.gz.sha512
shasum -a 512 -c elasticsearch-7.12.1-linux-x86_64.tar.gz.sha512 
tar -xzf elasticsearch-7.12.1-linux-x86_64.tar.gz
cd elasticsearch-7.12.1/ 

启动命令：./elasticsearch -d
* 错误1: can not run elasticsearch as root 
解决方法：
	创建es用户组及es用户
	groupadd es
	useradd es -g es
	passwo es #p:caiyisb
	更改es文件夹及内部文件的所属用户及组为es:es（在elasticsearch根目录执行此命令）
	chown -R es:es elasticsearch-7.12.1
	切换到es用户再启动
	su es
	./elasticsearch -d
	访问IP:9200即可查看elasticsearch基本信息
* 外网访问错误2：
解决方法：修改network配置
	vim elasticsearch-7.12.1/config/elasticsearch.yml
	修改为network.host: 0.0.0.0
* 错误3：max virtual memory areas vm.max_map_count [65530] is too low, increase to at least [262144]
解决方法：执行命令
	sudo sysctl -w vm.max_map_count=2621441
* 错误4：bootstrap checks failed. You must address the points described in the following [2] lines before starting Elasticsearch
解决方法：修改cluster
	vim elasticsearch-7.12.1/config/elasticsearch.yml
	修改为cluster.initial_master_nodes: ["node-1"]

# kibana 安装
curl -O https://artifacts.elastic.co/downloads/kibana/kibana-7.12.1-linux-x86_64.tar.gz
curl https://artifacts.elastic.co/downloads/kibana/kibana-7.12.1-linux-x86_64.tar.gz.sha512 | shasum -a 512 -c - 
tar -xzf kibana-7.12.1-linux-x86_64.tar.gz
cd kibana-7.12.1-linux-x86_64/ 

> 同样需要切换到es用户启动
su es
./bin/kibana &

* 外网访问错误1：
解决方法：config/kibana.yml
	修改为server.host: "0.0.0.0"
	
# es 简单curd操作
> 基于HTTP协议，以JSON为数据交互格式的RESTful API.
向Elasticsearch发出的请求的组成部分与其它普通的HTTP请求是一样的：
```
curl -X<VERB> '<PROTOCOL>://<HOST>:<PORT>/<PATH>?<QUERY_STRING>' -d '<BODY>'
```
### 使用postman操作Elasticsearch RESTful API
* create index
```
PUT
http://120.78.*.*:9200/test
Request Headers
content-type: application/json; charset=UTF-8
Bodyraw (json)
{
  "mappings": {
    "properties": {
      "id": {
        "type": "keyword"
      },
      "organization": {
        "type": "keyword"
      }
    }
  }
}

```
* add data
```
POST
http://120.78.*.*:9200/test/_doc/2
Request Headers
content-type: application/json; charset=UTF-8
Bodyraw (json)
{
  "id": 2,
  "organization": "A0001"
}
```
* delete index
```
DEL
http://120.78.*.*:9200/test
Request Headers
content-type: application/json; charset=UTF-8
Bodyraw (json)
None
```
* _delete_by_query data
```
POST
http://120.78.*.*:9200/test/_delete_by_query
Request Headers
content-type: application/json; charset=UTF-8
Bodyraw (json)
{
  "query": {
    "match": {
      "id": "2"
    }
  }
}
```