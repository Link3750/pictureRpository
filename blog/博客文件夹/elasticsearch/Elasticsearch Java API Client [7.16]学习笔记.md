# Elasticsearch Java API Client [7.16]学习笔记

## 官方文档

https://www.elastic.co/guide/en/elasticsearch/client/java-api-client/current/index.html

## api介绍

*该处主要是翻译自官方文档，如有缺陷敬请指出*

* Elasticsearch Java API Client（EJAC）是elasticsearch 官方提供的最新版本java API。里面包含了所有的elasticsearch APIs
* 为所有的EJAC提供了阻塞、异步执行的方法
* 由于使用了构造者模式以及函数式编程，使得程序员在构建复杂的查询语句时能够同时兼顾可读性与简明性
* api中使用了Jackson，确保查询出来的语句能直接转换成项目中的类，而不需要使用人员再手动反序列化。
* 使用Java Low Level REST Client 创建http客户端，该客户端负责处理所有传输级别的问题：HTTP 连接池、重试、节点发现等。

## 引入依赖

*要求jdk8及以后版本*

```xml
<project>
  <dependencies>
    <dependency>
      <groupId>co.elastic.clients</groupId>
      <artifactId>elasticsearch-java</artifactId>
      <version>7.16.3</version>
    </dependency>
    <dependency>
      <groupId>com.fasterxml.jackson.core</groupId>
      <artifactId>jackson-databind</artifactId>
      <version>2.12.3</version>
    </dependency>
  </dependencies>
</project>
```

## 连接es

``` java
// 创建 low-level 客户端
RestClient restClient = RestClient.builder(
    new HttpHost("localhost", 9200)).build();

// 利用Jackson mapper将low-level客户端转成transport类
ElasticsearchTransport transport = new RestClientTransport(
    restClient, new JacksonJsonpMapper());

// 创建EJAC客户端连接
ElasticsearchClient client = new ElasticsearchClient(transport);
```

## 一个简单的查询请求

``` java
//利用lambda表达式创建各种查询要求
SearchResponse<Product> search = client.search(s -> s // search request
    .index("products") //索引
    .query(q -> q //查询语句
        .term(t -> t
            .field("name") // 字段名
            .value(v -> v.stringValue("bicycle")) // 字段值
        )),
    Product.class); // 将查询出的结果转换成实体类

// 遍历结果，进行其他操作
for (Hit<Product> hit: search.hits().hits()) {
    doSomething(hit.source());
}
```

*以上都是官网提供的简单教程，实际操作还需要自行摸索，以下为我学习过程中编写的代码*

## 匹配多个字段的搜索条件

``` java
//匹配字段1
Query query1 = Query.of(q -> q
                          .match(m -> m
                                 .field("column1")
                                 .query(mq -> mq
                                        .stringValue("keyword1"))));
// 匹配字段2
Query query2 = Query.of(q -> q
                          .match(m -> m
                                 .field("column2")
                                 .query(mq -> mq
                                        .stringValue("keyword2"))));
//构建一个查询条件数组
List<Query> queryList = new ArrayList<>();
queryList.add(query1);
queryList.add(query2);
//构建查询请求
SearchRequest searchRequest = SearchRequest.of(r -> r
                .index("index1")
              // 可以将请求全写进去
              //.query(q -> q.bool(b -> b.must(query1, query2)))
              //也可以使用一个请求数组，将数组传进must里面
                .query(q -> q.bool(b -> b.must(queryList)))
              // 一些较为简单的请求甚至还可以直接在里面写lambda表达式
              //.query(q -> q.bool(b -> b.must(m -> m.match([lambda functions]))))
                .size(10));
//查询
SearchResponse<Entity> searchResponse = null;
try {
    searchResponse = client.search(searchRequest, Entity.class);
} catch (IOException e) {
    log.error("搜索es出现问题", e);
}
//进行其他业务逻辑
doSomething(searchResponse);
```

## 创建索引

```java
ElasticsearchClient client = ElasticClientQueryUtils.initEsConnection();
Map<String, Property> propertyMap = new HashMap<>(10);
propertyMap.put("code", Property.of(p -> p.keyword(k -> k.boost(1.0))));
// 创建索引的请求
co.elastic.clients.elasticsearch.indices.CreateIndexRequest createIndexRequest = co.elastic.clients.elasticsearch.indices.CreateIndexRequest.of(cir -> cir
        .index("publish_demo")
        .settings(is -> is
                .analysis(a -> a
                        .normalizer("lowercase_normalizer", n -> n
                                .custom(c -> c
                                        .filter("lowercase")))))
        .mappings(m -> m.properties(propertyMap)));
// 通过client的indices方法去创建索引
co.elastic.clients.elasticsearch.indices.CreateIndexResponse createIndexResponse = client.indices().create(createIndexRequest);
```

但是新的api似乎不支持直接输入json串去创建索引（至少我还没找到）

而且我们有个业务需求，需要调低搜索词出现频度对于的es打分的影响,需要在settings中添加如下配置：

``` json
"similarity": {
  "my_bm25": {
    "type": "BM25",
      // b 用来控制文档长度对权值的惩罚程度。ES 中默认 b=0.75
      "b": "0.75",
      // k1：衡量高频term所在文档和低频term所在文档相关性的差异
      "k1": "0.3"
  }
}
```

但似乎无法在现有api提供的接口中实现。

## 与原版API优缺点的比较

1. EJAC 引入了函数式编程，使用起来更加灵活，ERHLC需要构建相当多的查询对象。
2. 对于习惯了函数式编程以及对es api熟悉的人来说，可读性有较大的提升，根据代码就能很快知道语句是如何编写的。

1. 由于现有版本EJAC创建请求时无法构建中间步骤，因此代码编写起来有一定难度，且会降低可读性
2. EJAC无法传入现成的json字符串
