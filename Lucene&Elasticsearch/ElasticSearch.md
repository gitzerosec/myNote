## 一、ElasticSearch介绍

### 1.1.什么是ES

ElasticSearch简称es，是基于Lucene的一款**分布式全文检索服务器**，实时存储，检索数据，处理PB级别数据，扩展性好是它的几大特点。es也使用Java开发，底层为Lucene来实现索引搜索功能，但是它使用**RESTful风格API**来简化Lucene复杂的操作，从而使得全文检索更简单。

### 1.2.ES的使用场景

所有用到搜索的地方都有ES的影子，如：

github的搜索功能，github自2013年起使用ES来作为搜索引擎，来处理PB级别的数据。

维基百科：启动以elasticsearch为基础的核心搜索架构

SoundCloud：“SoundCloud使用ElasticSearch为1.8亿用户提供即时而精准的音乐搜索服务”

百度：百度目前广泛使用ElasticSearch作为文本数据分析，采集百度所有服务器上的各类指标数据及用户自
定义数据，通过对各种数据进行多维分析展示，辅助定位分析实例异常或业务层面异常。目前覆盖百度内部
20多个业务线（包括casio、云分析、网盟、预测、文库、直达号、钱包、风控等），单集群最大100台机
器，200个ES节点，每天导入30TB+数据

新浪使用ES 分析处理32亿条实时日志

阿里使用ES 构建挖财自己的日志采集和分析体系 

### 1.3.ES对比Solr

- solr使用zookeeper作为分布式管理，而ES本身自带分布式协调管理功能
- solr支持多种格式数据，es只支持json格式数据
- solr在传统搜索中强于es，es在实时搜索效率高于solr
- solr本身官方扩展功能多，es更注重核心，扩展功能由第三方插件提供

## 二、ElasticSearch概念

ES是面向文档操作，指存储整个文档且对其索引，使之快速搜索文档内容，ES对比关系型数据库：

MYSQL		 ‐> Databases ‐> Tables ‐> Rows	     ‐> Columns
Elasticsearch  ‐> Indices	  ‐> Types  ‐> Documents ‐> Fields 

### 2.1.索引 index

ES中的索引指索引库，类似关系型数据库中的一个数据库Database，ES服务器可以有多个索引，保存不同特征的文档集合，例如用户的文档，产品的文档，订单的文档等。

### 2.2.类型 type

ES中的类型就好比MYSQL中的表，一个索引用有多个类型，比如一个索引存储游戏数据，而其中又有lol数据，dota数据，这样就定义lol数据为一个类型，dota数据为另一个类型。

### 2.3.文档 document

ES中的文档就好比MYSQL中的一条记录，是索引的基础信息单元，文档由json形式保存，每个type可以存储多个文档。

### 2.4.字段 field

ES中的字段也对应MYSQL中的字段，根据文档数据的不同属性来区分的分类标识。如文档名称，内容，路径，大小等。

### 2.5.映射 mapping

映射类似于MYSQL中的表结构的定义，用来限制数据方式规则，如某个字段的数据类型，默认值，分词器，是否索引等。好的映射关系可以提高数据处理的性能。

### 2.6.集群 cluster

集群就是由多个ES服务器节点组成的，共同拥有数据，索引，一起提高搜索功能。一个集群只有一个名字，如“elasticSearch”，每个节点设置相同的集群名字标识他们在一个集群里。

### 2.7. 节点 node

一个节点对应一个服务器，在一个集群大家庭里，参与搜索与索引功能。每个节点有不同的名字，这个名字一般是随机的。

### 2.8.分片和复制

分片是ES数据存储的文件块，是**数据的最小单元块**，对于一个10亿数据的文档而言，如果仅仅有一个节点来存储，那样是无法实现的，而且就算实现了对其操作也会极其缓慢，所以ES提供了分片机制，当创建索引时确定分片数量，例如分了5片，那么存储数据时ES会均衡的将数据分在这5片中，而不是同一个。

而对于复制是为了防止服务器故障而导致数据丢失，对于5个主分片会进行拷贝，形成5个从分片提供数据的冗余。

索引默认创建5个分片和一组从分片，**只有ES形成集群时，从分片才会被启用**，因为主从在一个ES节点上是没有意义的，ES故障会导致主从都损坏。

## 三、ElasticSearch的安装

安装就不详细说了，百度一大把，而且很简单，windows平台下：

官网下载ES压缩包，解压缩运行/bin/elasticsearch.bat即可，localhost:9200即可查询是否启动成功

图形化界面使用elasticsearch-head，基于nodejs，需要安装nodejs以及npm安装grunt，安装好后在head-master目录下`npm install`，再运行`grunt server`即可，输入localhost:9100即可查看

需要注意es跨域问题，在es的conf/elasticsearch.yml中配置：

```yml
http.cors.enabled: true
http.cors.allow‐origin: "*" 
```

## 四、ElasticSearch客户端操作

这里我们使用Postman工具来对ES索引进行操作

因为ES是RESTful风格，所以PUT—增加，POST—修改，DELETE—删除，GET—查询

### 4.1.创建索引与mappings

在我们创建索引的时候，可以直接创建一个没有mappings的索引，也可以带着mappings一起创建

1）只创建索引

![1570605268369](C:\Users\S1\AppData\Roaming\Typora\typora-user-images\1570605268369.png)

这里我们在请求参数json中没有添加任何数据，即创建一个空的索引test

2）创建索引与mappings

![1570605138300](C:\Users\S1\AppData\Roaming\Typora\typora-user-images\1570605138300.png)

这次我们在请求体中添加了mappings，下一层的article是type类型，即数据库中的表名，在下一层properties又包含了字段类型，type为长整型long或text文本，store表示是否存储，index是否建立索引，analyzer使用哪种解析器。

### 4.2.单独设置mappings

当然我们创建空的没有结构索引后，也可以后期为其添加结构映射mappings

![1570605361068](C:\Users\S1\AppData\Roaming\Typora\typora-user-images\1570605361068.png)

URL为`IP:端口/索引名/类型/_mappings`，请求体中就是其字段类型

### 4.3.删除索引

![1570605903840](C:\Users\S1\AppData\Roaming\Typora\typora-user-images\1570605903840.png)

因为是RESTful风格，所以DELETE请求即可

### 4.4.创建Document

![1570606810590](C:\Users\S1\AppData\Roaming\Typora\typora-user-images\1570606810590.png)

创建document时，URL为`IP:端口/索引名/类型/文档主键ID`，请求体为设置的mapping映射字段。

这里会发现字段id和主键id是一样的1，其实这是两个不同的东西，一个是es文档主键id，一个是文档字段id，即可以不存在id这个字段，但是不能少了主键ID，如果请求URL上不设置ID，ES会自动赋予随机的ID，一般来说主键ID和字段ID设置成一致的。

### 4.5.删除Document

![1570607233171](C:\Users\S1\AppData\Roaming\Typora\typora-user-images\1570607233171.png)

URL：`IP:端口/索引名/类型/主键ID`，注意这里的ID是主键ID而不是字段ID

### 4.6.修改Document

![1570607442608](C:\Users\S1\AppData\Roaming\Typora\typora-user-images\1570607442608.png)

URL：`IP:端口/索引名/类型/主键ID`，请求体为文档内容。这里ES基于Lucene，所以底层也是先删除再新增的操作。

### 4.7.查询索引

查询有三种，根据id，关键字，QueryString查询

#### 4.7.1.根据ID查询

![1570608523919](C:\Users\S1\AppData\Roaming\Typora\typora-user-images\1570608523919.png)

只需get方法，URL为`IP:端口/索引名/类型/主键ID`

这种查询只能查出一条数据。

#### 4.7.2.根据关键字查询

![1570608608924](C:\Users\S1\AppData\Roaming\Typora\typora-user-images\1570608608924.png)

使用POST请求，URL为`IP:端口/索引名/类型/_search`，请求体中需要拼接query与term

![1570608664383](C:\Users\S1\AppData\Roaming\Typora\typora-user-images\1570608664383.png)

可以查出多条数据

#### 4.7.3.QueryString查询

在Lucene中具有QueryPaser查询，是先将请求关键词进行分词再进行查询，再ES中当然也支持，但是改名叫QueryString了。

![1570609286169](C:\Users\S1\AppData\Roaming\Typora\typora-user-images\1570609286169.png)

URL和普通关键词查询相同，在请求体中`query`下层是`query_string`然后固定内容为`default_field`请求字段和`query`关键词

![1570609362029](C:\Users\S1\AppData\Roaming\Typora\typora-user-images\1570609362029.png)

可以看到命中了document或content的数据

## 五、ElasticSearch的分析器

### 5.1.传统Standard分析器

和Lucene一样，ES的分析器默认使用Standard，只对英文友好，对中文没有分析效果

![1570610868059](C:\Users\S1\AppData\Roaming\Typora\typora-user-images\1570610868059.png)

URL为`IP:端口/_analyzer?analyzer=分析器名称&text=分析内容`

可以看到对中文是一个字一个字分析。所以还是需要使用IK分析器。

### 5.2.IK分析器

只需到ik github上下载相同版本压缩包后，解压缩放到es的plugins文件夹下，然后重启服务即可。

ik分析器又分为两种，一是`ik_smart`指最少划分，一是`ik_max_word`指最细粒度划分，下图可以看到区别：

![1570623480119](C:\Users\S1\AppData\Roaming\Typora\typora-user-images\1570623480119.png)

![1570623463217](C:\Users\S1\AppData\Roaming\Typora\typora-user-images\1570623463217.png)

## 六、ElasticSearch集群

ES集群的概念件见第二节，就不过多介绍了，概念了解即可，对于更深层的ES集群的底层实现后面在进行学习。

这里我们主要学习下ES集群的搭建与测试。

三节点集群搭建：

ES文件夹复制三份，配置config/elasticsearch.yml配置文件，添加：

```yml
#节点3配置：
#集群名称，唯一
cluster.name: my-elasticsearch
#节点名称，必须不一样
node.name: node-3
#本机ip
network.host: 127.0.0.1
#服务端口
http.port: 9203
#集群通信端口
transport.tcp.port: 9303
#集群自动发现机器ip集合
discovery.zen.ping.unicast.hosts: ["127.0.0.1:9301","127.0.0.1:9302","127.0.0.1:9303"]
```

为三个配置文件配置好，然后顺序启动。在es可视化工具中可以看到，3节点连成集群

![1570670076366](C:\Users\S1\AppData\Roaming\Typora\typora-user-images\1570670076366.png)

创建索引：`PUT 127.0.0.1:9200/test`

![1570670282795](C:\Users\S1\AppData\Roaming\Typora\typora-user-images\1570670282795.png)

这时我们发现分片副本没有分配节点，健康值还是像原来的单节点ES一样，网上查了一下，是因为数据保存磁盘默认值为85%，如果磁盘占用量大于85%，便不会自动分配副本，这时我们有两种方法：

1）设置磁盘占用率为90%

URL:`PUT 127.0.0.1:9200/_cluster/settings`

请求体：

```yml
{
	"transient":{
		"cluster.routing.allocation.disk.watermark.low" : "90%"
	}
}
```

2）设置data存放位置

在配置文件elasticsearch.yml中配置

![1570670648628](C:\Users\S1\AppData\Roaming\Typora\typora-user-images\1570670648628.png)

也可以解决此问题

![1570672858014](C:\Users\S1\AppData\Roaming\Typora\typora-user-images\1570672858014.png)

正确的集群节点分片，粗框框的是主分片，细一点的是分片备份