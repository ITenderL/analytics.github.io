# 基于Docker学习ElasticSearch

## 1. 环境搭建

### 1.1. 安装ElasticSearch

```bash
# 拉取镜像
docker pull elasticsearch:7.6.0
# 运行es容器
docker run -d --name itender_es -p 9200:9200 -p 9300:9300 -e ES_JAVA_OPTS="-Xms512m -Xmx512m" -e "discovery.type=single-node" elasticsearch:7.6.0
# 查看
curl http://localhost:9200
http://172.18.169.102:9200/
```

### 1.2.安装kibana

```bash
# 拉取镜像
docker pull kibana:7.6.0
# 运行容器
docker run -d --name=itender_kibana -p 5601:5601 kibana:7.6.0
```

### 1.3.Kibana连接ES

#### 1.3.1. 普通方式

```bash
# 进入kibana容器
docker exec -it 82d41cee7e64 bash
# 进入config目录
cd config
# 修改kibana.yml文件
vi kibana.yml

server.name: kibana
server.host: "0"
elasticsearch.hosts: [ "http://172.18.169.102:9200" ] # ip为es的宿主机ip
xpack.monitoring.ui.container.elasticsearch.enabled: true

# 退出容器，重启容器
docker restart 'kibana容器'

```

#### 1.3.2. 目录挂载方式

```bash
 # 复制容器中kibana.yml 到宿主机的/elk/kibana/文件夹下
 docker cp '82d41cee7e64容器名':'/usr/share/kibana/config/kibana.yml容器的文件目录' /elk/kibana/
 # 重新运行kibana容器，目录挂载的方式
 docker run -d --name=kibana -p 5601:5601 -v /elk/kibana/kibana.yml:/usr/share/kibana/config/kibana.yml kibana:7.6.0
 
```

### 1.4. Docker-compose安装

#### 4.1. 安装docker-compose

```bash
# 第一种方式：安装docker-compose
sudo curl -L https://github.com/docker/compose/releases/download/1.16.1/docker-compose-`uname -s`-`uname -m` -o /usr/local/bin/docker-compose
# 第二种方式：安装docker-compose
sudo curl -L https://get.daocloud.io/docker/compose/releases/download/1.25.1/docker-compose-`uname -s`-`uname -m` -o /usr/local/bin/docker-compose
# 添加可执行权限
sudo chmod +x /usr/local/bin/docker-compose
# 查看是否安装成功
```

#### 4.2. 创建docker-compose文件

```yml
version: "2.2"
volumes:
  data:
  config:
  plugin:
networks:
  es:
services:
  elasticsearch:
    image: elasticsearch:7.6.0
    ports:
      - "9200:9200"
      - "9300:9300"
    networks:
      - "es"
    environment:
      - "discovery.type=single-node"
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
    volumes:
      - data:/usr/share/elasticsearch/data
      - config:/usr/share/elasticsearch/config
      - plugin:/usr/share/elasticsearch/plugins
  kibana:
    image: kibana:7.6.0
    ports:
      - "5601:5601"
    networks:
      - "es"
    volumes:
      - /elk/es-kibana-compose/kibana.yml:/usr/share/kibana/config/kibana.yml
```

```yml
server.name: kibana
server.host: "0"
elasticsearch.hosts: [ "http://elasticsearch:9200" ]
xpack.monitoring.ui.container.elasticsearch.enabled: true
```

#### 4.3. 启动es和kibana

```bash
# 启动es和kibana
docker-compose up -d
# 查看日志
docker logs -f "es容器"
docker logs -f "kibana容器"
```

### 1.5 ik分词器安装

修改docker-compose.yml文件，进行文件映射

```bash
# 现在ik分词器，这里要注意的是ik分词器的版本要与es版本保持一致
wget https://github.com/medcl/elasticsearch-analysis-ik/releases/download/v7.6.0/elasticsearch-analysis-ik-7.6.0.zip
```

```bash
# 关闭容器
docker-compose down
```

修改docker-compose.yml

```bash
version: "2.2"
volumes:
  data:
  config:
networks:
  es:
services:
  elasticsearch:
    image: elasticsearch:7.6.0
    ports:
      - "9200:9200"
      - "9300:9300"
    networks:
      - "es"
    environment:
      - "discovery.type=single-node"
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
    volumes:
      - data:/usr/share/elasticsearch/data
      - config:/usr/share/elasticsearch/config
      - /elk/es-kibana-compose/ik-7.6.0:/usr/share/elasticsearch/plugins/ik-7.6.0
  kibana:
    image: kibana:7.6.0
    ports:
      - "5601:5601"
    networks:
      - "es"
    volumes:
      - /elk/es-kibana-compose/kibana.yml:/usr/share/kibana/config/kibana.yml
```

## 2. 几个核心概念

### 索引 index

一个索引就是一个拥有几分相似特征的文档的集合。比如说，你可以有一个客户数据的索引，另一个产品目录的索引，还有一个订单数据的索引。一个索引由一个名字来标识（必须全部是小写字母的），并且当我们要对对应于这 个索引中的文档进行索引、搜索、更新和删除的时候，都要使用到这个名字。在一个集群中，可以定义任意多的索引。

### 映射 mapping

mapping是处理数据的方式和规则方面做一些限制，如某个字段的数据类型、默认值、分析器、是否被索引等等， 这些都是映射里面可以设置的，其它就是处理es里面数据的一些使用规则设置也叫做映射，按着最优规则处理数据对性能提高很大，因此才需要建立映射，并且需要思考如何建立映射才能对性能更好。 

### 文档  document

一个文档是一个可被索引的基础信息单元。比如，你可以拥有某一个客户的文档，某一个产品的一个文档，当然， 也可以拥有某个订单的一个文档。文档以JSON（Javascript Object Notation）格式来表示，而JSON是一个到处存在的互联网数据交互格式。在一个index/type里面，你可以存储任意多的文档。注意，尽管一个文档，物理上存在于一个索引之中，文档必须 被索引/赋予一个索引的type。 

### 字段 Field

相当于是数据表的字段，对文档数据根据不同属性进行的分类标识 

# Elasticsearch集群搭建
## 1. 相关概念
**集群<cluster>**
一个集群就是由一个或多个节点组织在一起，它们共同持有整个的数据，并一起提供索引和搜索功能。一个集群由一个唯一的名字标识，这个名字默认就是“elasticsearch”。这个名字是重要的，因为一个节点只能通过指定某个集群的名字，来加入这个集群
**节点<Node>**
一个节点是集群中的一个服务器，作为集群的一部分，它存储数据，参与集群的索引和搜索功能。和集群类似，一个节点也是由一个名字来标识的，默认情况下，这个名字是一个随机的漫威漫画角色的名字，这个名字会在启动的时候赋予节点。这个名字对于管理工作来说挺重要的，因为在这个管理过程中，你会去确定网络中的哪些服务器对应于Elasticsearch集群中的哪些节点。

一个节点可以通过配置集群名称的方式来加入一个指定的集群。默认情况下，每个节点都会被安排加入到一个叫做“elasticsearch”的集群中，这意味着，如果你在你的网络中启动了若干个节点，并假定它们能够相互发现彼此，它们将会自动地形成并加入到一个叫做“elasticsearch”的集群中。
在一个集群里，只要你想，可以拥有任意多个节点。而且，如果当前你的网络中没有运行任何Elasticsearch节点，这时启动一个节点，会默认创建并加入一个叫做“elasticsearch”的集群。
**分片<Shards>&复制<replicas>**
一个索引可以存储超出单个结点硬件限制的大量数据。比如，一个具有10亿文档的索引占据1TB的磁盘空间，而任一节点都没有这样大的磁盘空间；或者单个节点处理搜索请求，响应太慢。为了解决这个问题，Elasticsearch提供了将索引划分成多份的能力，这些份就叫做分片。当你创建一个索引的时候，你可以指定你想要的分片的数量。每个分片本身也是一个功能完善并且独立的“索引”，这个“索引”可以被放置到集群中的任何节点上。分片很重要，主要有两方面的原因：
1）允许你水平分割/扩展你的内容容量。
2）允许你在分片（潜在地，位于多个节点上）之上进行分布式的、并行的操作，进而提高性能/吞吐量。

至于一个分片怎样分布，它的文档怎样聚合回搜索请求，是完全由Elasticsearch管理的，对于作为用户的你来说，这些都是透明的。

在一个网络/云的环境里，失败随时都可能发生，在某个分片/节点不知怎么的就处于离线状态，或者由于任何原因消失了，这种情况下，有一个故障转移机制是非常有用并且是强烈推荐的。为此目的，Elasticsearch允许你创建分片的一份或多份拷贝，这些拷贝叫做复制分片，或者直接叫复制。

复制之所以重要，有两个主要原因： 在分片/节点失败的情况下，提供了高可用性。因为这个原因，注意到复制分片从不与原/主要（original/primary）分片置于同一节点上是非常重要的。扩展你的搜索量/吞吐量，因为搜索可以在所有的复制上并行运行。总之，每个索引可以被分成多个分片。一个索引也可以被复制0次（意思是没有复制）或多次。一旦复制了，每个索引就有了主分片（作为复制源的原来的分片）和复制分片（主分片的拷贝）之别。分片和复制的数量可以在索引创建的时候指定。在索引创建之后，你可以在任何时候动态地改变复制的数量，但是你事后不能改变分片的数量。

默认情况下，Elasticsearch中的每个索引被分片5个主分片和1个复制，这意味着，如果你的集群中至少有两个节点，你的索引将会有5个主分片和另外5个复制分片（1个完全拷贝），这样的话每个索引总共就有10个分片。
## 2. Elasticsearch集群搭建
**集群规划，采用三个节点**
```bash
# 准备3个es节点 es 9200 9300
- web 9201 tcp:9301 node-1 elasticsearch.yml
- web 9202 tcp:9302 node-2 elasticsearch.yml
- web 9203 tcp:9303 node-3 elasticsearch.yml
```
 - 注意
      - 所有节点集群名称必须保持一致cluster.name
      - 每个节点必须有唯一的名字 node.name
      - 开启每个节点的远程连接network.host:0.0.0.0
      - 指定IP地址进行集群节点通信network_publish_host:
      - 修改web端口tcp端口http.port:transport.tcp.port
      - 指定集群中所有系节点通信列表discovery.seed_hosts:node-1 node-2 node-3相同
      - 允许集群初始化master节点节点数：cluster.initial_master_nodes:["node-1", "node-2", "node-3"]
      - 集群最少几个节点可用gateway.recover_after_nodes:2
      - 开启每个节点的跨域访问http.cors.enabled:true	 http.cors.allow.-origin:"*"

**配置文件**

```yml
cluster
	node-1
		config/elasticsearch.yml
			# 集群名称
			cluster.name: es-cluster 
			
			#节点名称
			node.name: node-1 
			
			# 发布地址，一个单一地址，用于通知集群中的其他节点，以便其他的节点能够和它通信。当前，一个 elasticsearch 节点可能被绑定到多个地址，但是仅仅有一个发布地址
			# docker宿主机ip
			network.publish_host: 172.30.38.46
			
			# 开放远程连接，bind_host和publish_host一起设置
			network.host: 0.0.0.0 
			
			# 对外暴露的http请求端口
			http.port: 9201 
			
			# 集群节点之间通信用的TCP端口
			transport.tcp.port: 9301
			
			# 一个集群中最小主节点个数（防止脑裂，一般为n/2 + 1，n为集群节点个数）（7.10.1版本已取消？）
			discovery.zen.minimum_master_nodes: 2 
			
			# 新节点启动时能被发现的节点列表（新增节点需要添加自身）
			discovery.zen.ping.unicast.hosts: ["172.30.38.46:9301","172.30.38.46:9302","172.30.38.46:9303"]
			
			# 集群初始话指定主节点（节点名）,7版本必须设置
			cluster.initial_master_nodes: ["node-1","node-2","node-3"]
			
			# 跨域问题解决
			http.cors.enabled: true
			http.cors.allow-origin: "*"
	node-2
		config/elasticsearch.yml
			# 集群名称
			cluster.name: es-cluster 
			
			#节点名称
			node.name: node-1 
			
			# 发布地址，一个单一地址，用于通知集群中的其他节点，以便其他的节点能够和它通信。当前，一个 elasticsearch 节点可能被绑定到多个地址，但是仅仅有一个发布地址
			network.publish_host: 172.30.38.46
			
			# 开放远程连接，bind_host和publish_host一起设置
			network.host: 0.0.0.0 
			
			# 对外暴露的http请求端口
			http.port: 9202 
			
			# 集群节点之间通信用的TCP端口
			transport.tcp.port: 9302
			
			# 一个集群中最小主节点个数（防止脑裂，一般为n/2 + 1，n为集群节点个数）（7.10.1版本已取消？）
			discovery.zen.minimum_master_nodes: 2 
			
			# 新节点启动时能被发现的节点列表（新增节点需要添加自身）
			discovery.zen.ping.unicast.hosts: ["172.30.38.46:9301","172.30.38.46:9302","172.30.38.46:9303"]
			
			# 集群初始话指定主节点（节点名）,7版本必须设置
			cluster.initial_master_nodes: ["node-1","node-2","node-3"]
			
			# 跨域问题解决
			http.cors.enabled: true
			http.cors.allow-origin: "*"	
	node-3
		config/elasticsearch.yml
			# 集群名称
			cluster.name: es-cluster 
			
			#节点名称
			node.name: node-1 
			
			# 发布地址，一个单一地址，用于通知集群中的其他节点，以便其他的节点能够和它通信。当前，一个 elasticsearch 节点可能被绑定到多个地址，但是仅仅有一个发布地址
			network.publish_host: 172.30.38.46
			
			# 开放远程连接，bind_host和publish_host一起设置
			network.host: 0.0.0.0 
			
			# 对外暴露的http请求端口
			http.port: 9203 
			
			# 集群节点之间通信用的TCP端口
			transport.tcp.port: 9303
			
			# 一个集群中最小主节点个数（防止脑裂，一般为n/2 + 1，n为集群节点个数）（7.10.1版本已取消？）
			discovery.zen.minimum_master_nodes: 2 
			
			# 新节点启动时能被发现的节点列表（新增节点需要添加自身）
			discovery.zen.ping.unicast.hosts: ["172.30.38.46:9301","172.30.38.46:9302","172.30.38.46:9303"]
			
			# 集群初始话指定主节点（节点名）,7版本必须设置
			cluster.initial_master_nodes: ["node-1","node-2","node-3"]
			
			# 跨域问题解决
			http.cors.enabled: true
			http.cors.allow-origin: "*"	
```

创建/elk/escluster-kibana-compose/node-1，/elk/escluster-kibana-compose/node-2，/elk/escluster-kibana-compose/node-3文件夹。分别在三个文件夹下创建config文件夹并在config文件夹下创建`elasticsearch.yml`文件，

```yml
cluster
	node-1
		config/elasticsearch.yml
			cluster.name: es-cluster 
			node.name: node-1 
			network.publish_host: 172.30.38.46
			network.host: 0.0.0.0 
			http.port: 9201 
			transport.tcp.port: 9301
			discovery.zen.minimum_master_nodes: 2 
			discovery.zen.ping.unicast.hosts: ["172.30.38.46:9301","172.30.38.46:9302","172.30.38.46:9303"]
			cluster.initial_master_nodes: ["node-1","node-2","node-3"]
			http.cors.enabled: true
			http.cors.allow-origin: "*"
	node-2
		config/elasticsearch.yml
			cluster.name: es-cluster 
			node.name: node-2 
			network.publish_host: 172.30.38.46
			network.host: 0.0.0.0 
			http.port: 9202 
			transport.tcp.port: 9302
			discovery.zen.minimum_master_nodes: 2 
			discovery.zen.ping.unicast.hosts: ["172.30.38.46:9301","172.30.38.46:9302","172.30.38.46:9303"]
			cluster.initial_master_nodes: ["node-1","node-2","node-3"]
			http.cors.enabled: true
			http.cors.allow-origin: "*"	
	node-3
		config/elasticsearch.yml
			cluster.name: es-cluster 
			node.name: node-3
			network.publish_host: 172.30.38.46
			network.host: 0.0.0.0 
			http.port: 9203 
			transport.tcp.port: 9303
			discovery.zen.minimum_master_nodes: 2 
			discovery.zen.ping.unicast.hosts: ["172.30.38.46:9301","172.30.38.46:9302","172.30.38.46:9303"]
			cluster.initial_master_nodes: ["node-1","node-2","node-3"]
			http.cors.enabled: true
			http.cors.allow-origin: "*"	
```

`/elk/escluster-kibana-compose/kibana.yml`

```yml
server.name: kibana
server.port: 5606
server.host: "0"
elasticsearch.hosts: [ "http://es01:9201","http://es02:9202","http://es03:9203" ]
xpack.monitoring.ui.container.elasticsearch.enabled: true
```

docker-compose.yml文件

```yml
version: "2"
networks:
  escluster:
services:
  es01:
    image: elasticsearch:7.6.0
    ports:
      - "9201:9201"
      - "9301:9301"
    networks:
      - "escluster"
    environment:
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
    volumes:
      - /elk/escluster-kibana-compose/node-1/data:/usr/share/elasticsearch/data
      - /elk/escluster-kibana-compose/node-1/config/elasticsearch.yml:/usr/share/elasticsearch/config/elasticsearch.yml
      - /elk/escluster-kibana-compose/node-1/plugins/ik-7.6.0:/usr/share/elasticsearch/plugins/ik-7.6.0
  es02:
    image: elasticsearch:7.6.0
    ports:
      - "9202:9202"
      - "9302:9302"
    networks:
      - "escluster"
    environment:
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
    volumes:
      - /elk/escluster-kibana-compose/node-2/data:/usr/share/elasticsearch/data
      - /elk/escluster-kibana-compose/node-2/config/elasticsearch.yml:/usr/share/elasticsearch/config/elasticsearch.yml
      - /elk/escluster-kibana-compose/node-2/plugins/ik-7.6.0:/usr/share/elasticsearch/plugins/ik-7.6.0
  es03:
    image: elasticsearch:7.6.0
    ports:
      - "9203:9203"
      - "9303:9303"
    networks:
      - "escluster"
    environment:
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
    volumes:
      - /elk/escluster-kibana-compose/node-3/data:/usr/share/elasticsearch/data
      - /elk/escluster-kibana-compose/node-3/config/elasticsearch.yml:/usr/share/elasticsearch/config/elasticsearch.yml
      - /elk/escluster-kibana-compose/node-3/plugins/ik-7.6.0:/usr/share/elasticsearch/plugins/ik-7.6.0
  kibana:
    image: kibana:7.6.0
    ports:
      - "5606:5606"
    networks:
      - "escluster"
    volumes:
      - /elk/escluster-kibana-compose/kibana.yml:/usr/share/kibana/config/kibana.yml
```

执行docker-compose

```bash
# 启动
docker-compose up -d
# 查看日志
docker-compose logs -f
```

启动完成之后分别访问

http://172.30.38.46:9201/

http://172.30.38.46:9202/

http://172.30.38.46:9203/

```json
{
"name": "node-1",
"cluster_name": "es-cluster",
"cluster_uuid": "gC1ZHP8uSOqnqof0-Rdlcg",
"version": {
"number": "7.6.0",
"build_flavor": "default",
"build_type": "docker",
"build_hash": "7f634e9f44834fbc12724506cc1da681b0c3b1e3",
"build_date": "2020-02-06T00:09:00.449973Z",
"build_snapshot": false,
"lucene_version": "8.4.0",
"minimum_wire_compatibility_version": "6.8.0",
"minimum_index_compatibility_version": "6.0.0-beta1"
},
"tagline": "You Know, for Search"
}


{
"name": "node-2",
"cluster_name": "es-cluster",
"cluster_uuid": "gC1ZHP8uSOqnqof0-Rdlcg",
"version": {
"number": "7.6.0",
"build_flavor": "default",
"build_type": "docker",
"build_hash": "7f634e9f44834fbc12724506cc1da681b0c3b1e3",
"build_date": "2020-02-06T00:09:00.449973Z",
"build_snapshot": false,
"lucene_version": "8.4.0",
"minimum_wire_compatibility_version": "6.8.0",
"minimum_index_compatibility_version": "6.0.0-beta1"
},
"tagline": "You Know, for Search"
}

{
"name": "node-3",
"cluster_name": "es-cluster",
"cluster_uuid": "gC1ZHP8uSOqnqof0-Rdlcg",
"version": {
"number": "7.6.0",
"build_flavor": "default",
"build_type": "docker",
"build_hash": "7f634e9f44834fbc12724506cc1da681b0c3b1e3",
"build_date": "2020-02-06T00:09:00.449973Z",
"build_snapshot": false,
"lucene_version": "8.4.0",
"minimum_wire_compatibility_version": "6.8.0",
"minimum_index_compatibility_version": "6.0.0-beta1"
},
"tagline": "You Know, for Search"
}
```

访问kibana

http://172.30.38.46:5606/