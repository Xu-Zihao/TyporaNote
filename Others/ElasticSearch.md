#### 基础概念

ElasticSearch 是分布式文档数据库，允许多台服务器协同工作，每台服务器可以运行多个实例。单个实例称为一个**节点**（node），一组节点构成一个**集群**（cluster）。分片是底层的工作单元，**文档**保存在分片内，分片又被分配到集群内的各个节点里，每个分片仅保存全部数据的一部分。

##### Cluster 集群

Cluster 也就是集群的意思。Elasticsearch 集群由一个或多个节点组成，可通过其集群名称进行标识。通常这个 Cluster 的名字是可以在 Elasticsearch 里的配置文件中设置的。在默认的情况下，如我们的 Elasticsearch 已经开始运行，那么它会自动生成一个叫做 “elasticsearch” 的集群。



##### node 节点

单个 Elasticsearch 实例。 在大多数环境中，每个节点都在单独的盒子或虚拟机上运行。一个集群由一个或多个 node 组成。在测试的环境中，我可以把多个 node 运行在一个 server 上。在实际的部署中，大多数情况还是需要一个 server 上运行一个 node。node上存放的是分片和备份。

node根据作用可以分为如下几种：

- master-eligible：可以作为主 node。一旦成为主 node，它可以管理整个 cluster 的设置及变化：创建，更新，删除 index；添加或删除 node；为 node 分配 shard

- data：数据 node

- ingest: 数据接入（比如 pipepline)

- machine learning (Gold/Platinum License)

- Coordinating node：严格来说，这个不是一个种类的节点。它甚至可以是上面的任何一种节点。这种节点通常是接受客户端的 HTTP 请

  一般而言，一个节点最好专注于一种功能。



##### Document(类似于mysql中的record)

Elasticsearch 是面向文档的，这意味着你索引或搜索的最小数据单元是文档。文档通常是数据的 [JSON](https://baike.baidu.com/item/JSON/2462549?fr=aladdin) 表示形式。

```xml
{
          "name": "Elasticsearch Denver",
          "organizer": "Lee",
          "location": "Denver, Colorado, USA"
       }
```



##### type 类型

类型是文档的逻辑容器，类似于表是行的容器。type定义了不同的映射模式，例如

```xml
{
 	"name": "zhangsan"
	"id": "18"
}
```

对于该文档中的id，type可以将其映射类型定义为string类型，也可以将其定义为数字类型，而这两种type最终会导致文档被不相同地解释。在8.0的版本中，type 被彻底删除。



##### index 索引

inde相当于mysql地数据库。在elasticsearch中，index会被分布在一个（默认）或多个shard（分片）中，而每个 shard 相应于一个 Aache Lucene 的 index。每个 Index 一个或许多的 documents 组成，所以这些 document 也就分布于不同的 shard 之中。

每当一个文档进来后，根据文档的 id 会自动进行 hash 计算，并存放于计算出来的 shard 实例中，这样的结果可以使得所有的 shard 都比较有均衡的存储，而不至于有的 shard 很忙。

```xml
shard_num = hash(_routing) % num_primary_shards
```

从上面的公式我们也可以看出来，我们的 shard 数目是不可以动态修改的，否则之后也找不到相应的 shard 号码了。而replica 的数目是可以动态修改的。



##### shard 分片

Elasticsearch 是一个分布式搜索引擎，Elasticsearch 提供了将索引划分成多份的能力，这些份就叫做分片（shard）。当你创建一个索引的时候，你可以指定你想要的分片(shard)的数量。每个分片本身也是一个功能完善并且独立的“索引”，这个“索引”可以被放置到集群中的任何节点上。所以，一个索引可以存储超出单个结点硬件限制的大量数据（毕竟一个索引可以存储在多个节点上）。

 Elasticsearch 自动管理这些分片的排列。Elasticsearch还会根据需要自动平衡分片。

分片的操作：

- 允许你水平分割/扩展你的内容容量
- 允许你在分片（潜在地，位于多个节点上）之上进行分布式的、并行的操作，进而提高性能/吞吐量

分片的类型：

- Primary shard: 每个文档都存储在一个Primary shard。 索引文档时，它首先在 Primary shard上编制索引，然后在此分片的所有副本上（replica）编制索引。索引可以包含一个或多个主分片。 此数字确定索引相对于索引数据大小的可伸缩性。 创建索引后，无法更改索引中的主分片数。
- Replica shard: 每个主分片可以具有零个或多个副本。 副本是主分片的副本，有两个目的：
  - 故障转移：如果主分片故障，可以将副本分片提升为主分片。即使你失去了一个 node，那么副本分片还是拥有所有的数据
  - 提高性能：get 和 search 请求可以由主 shard 或副本 shard 处理。

> 在正常的情况下，主分片和和该分片的副本存在同一个node是不允许的，因为如果这个节点出错的话，该分片的数据可能会永久丢失。







#### Elasticsearch Java Rest Client API



------

##### QueryBuilder

构建查询条件，即设置查询语句，包括查询的内容，查询的具体方法。

如：

```java
MatchQueryBuilder matchQueryBuilder = new MatchQueryBuilder("user", "kimchy"); //匹配user字段的值为kinchy的文档
```

`QueryBuilder` 创建后，就可以调用方法来配置它的查询选项：

```java
matchQueryBuilder.fuzziness(Fuzziness.AUTO);  // 模糊查询
matchQueryBuilder.prefixLength(3); // 前缀查询的长度
matchQueryBuilder.maxExpansions(10); // max expansion 选项，用来控制模糊查询
```

也可以使用`QueryBuilders` 工具类来创建 `QueryBuilder` 对象。这个类提供了函数式编程风格的各种方法用来快速创建 `QueryBuilder` 对象。

```java
QueryBuilder matchQueryBuilder = QueryBuilders.matchQuery("user", "kimchy")
                                        .fuzziness(Fuzziness.AUTO)
                                                .prefixLength(3)
                                                .maxExpansions(10);
```



------



##### SearchSourceBuilder

配置搜索行为，即设置如何搜索文档，再根据搜索的文档执行搜索行为，其中包括聚合、排序、过滤、搜索的位置和范围等。所以SearchSourceBuilder必须传入QueryBuilder作为参数

例子：

```java
SearchSourceBuilder sourceBuilder = new SearchSourceBuilder();  // 默认配置
sourceBuilder.query(QueryBuilders.termQuery("user", "kimchy")); // 设置搜索，可以是任何类型的 QueryBuilder
sourceBuilder.from(0); // 起始 index
sourceBuilder.size(5); // 大小 size
sourceBuilder.timeout(new TimeValue(60, TimeUnit.SECONDS)); // 设置搜索的超时时间
```



------



##### searchRequest

经过上面的两步，我们已经设置好如何查询以及如何执行查询，现在要做的当然就是执行查询了。searchRequest用来发送查询执行请求，相应的操作包括查询哪个索引（库），以及映射关系（即查询内容的值映射成的类型，type在es8.0后被弃用）。

基本的查询操作如下：

```java
SearchRequest searchRequest = new SearchRequest(); //设置查询细节
SearchSourceBuilder searchSourceBuilder = new SearchSourceBuilder(); //查询执行细节
searchSourceBuilder.query(QueryBuilders.matchAllQuery()); // 添加 match_all 查询
searchRequest.source(searchSourceBuilder); // 将 SearchSourceBuilder  添加到 SeachRequest 中
```

可选参数：

```java
SearchRequest searchRequest = new SearchRequest("index");  // 设置搜索的 index
searchRequest.types("type");  // 设置搜索的 type
```



------



##### SearchResponse

经过上面的三步，我们已经完整地设置好一个ES的查询请求，接下来要做的就是得到一个客户端，用这个客户端执行查询请求，接下来最重要的当然时拿到响应(SearchResponse)了，如下：

```java
RestHighLevelClient client = ElasticSearchUtil.getClient("name");
SearchResponse searchResponse = client.search(searchRequest, RequestOptions.DEFAULT);
SearchHits hits = searchResponse.getHits();
SearchHit[] searchHits = hits.getHits();
```

我们拿到了searchHits，即命中的原始数据，经过相应的反序列化就可以转化为我们可以使用的对象k





