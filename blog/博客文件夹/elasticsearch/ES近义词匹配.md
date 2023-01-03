# ES近义词匹配

ES近义词匹配搜索需要用户提供一张满足相应格式的近义词表，并在创建索引时设计将该表放入`settings`中。

近义词表的可以直接以字符串的形式写入`settings`中也可以放入文本文件中，由es读取。

## 近义词表格式

近义词表需要满足以下格式要求：

1. `A => B,C`格式
   * 这种格式在搜索时会将搜索词A替换成B、C，且B，C互不为同义词
   
2.  `A,B,C,D` 格式
   
   这种格式得分情况讨论：
   
   * 当`expand == true`时，这种格式等价于`A,B,C,D => A,B,C,D`即ABCD互为同义词
   
   * 当`expand == false`时，这种格式等价于A,B,C,D => A，即ABCD四个词在搜索时会被替换成A

## 如何使用近义词表进行查询

### 建立索引

```json
PUT /fond_goods
{
  "settings": {
    "number_of_replicas": 0,
    "number_of_shards": 1,
    "analysis": {
      "analyzer": {
        "my_whitespace":{
          "tokenizer":"whitespace",
          "filter": ["synonymous_filter"]
        }
      },
      "filter": {
        "synonymous_filter":{
          "type": "synonym",
          "expand": true
          "synonyms": [
            "A, B, C, D"
            ]
        }
      }
    }
  },
  "mappings": {
    "properties": {
      "code":{
        "type": "keyword"
      },
      "context":{
        "type": "text",
        "analyzer": "my_whitespace"
      },
      "color":{
        "type": "text",
        "analyzer": "my_whitespace"
      }
    }
  }
}
```

#### 参数解释

* `expand` 默认值为 `true` 。
* `lenient` 默认值为`false` 若`lenient`值为`true`， es会忽略转换近义词文件时的报错。值得注意的是，只有当遇到近义词无法转换时出现的异常才会被忽略掉，具体例子可以参考官网 [ https://www.elastic.co/guide/en/elasticsearch/reference/7.16/analysis-synonym-tokenfilter.html ]。
* `synonyms`近义词表，即开始所说要按格式填写的近义词表。
* `synonyms`也可替换成`synonyms_path`，此时需要填写一个外部文件的路径。该文件可以是某个外部的网页，也可以是存放在本地的文件。
* `format` 当该参数值为`wordnet`时，可以使用wordnet英文词汇数据库中的近义词。

## 使用案例

### 构建索引

``` json
PUT /fond_goods
{
  "settings": {
    "number_of_replicas": 0,
    "number_of_shards": 1,
    "analysis": {
      "analyzer": {
        "my_whitespace":{................................................................ I
          "tokenizer":"whitespace",
          "filter": ["synonymous_filter"]
        }
      },
      "filter": {
        "synonymous_filter":{
          "type": "synonym",
          "synonyms_path": "synonym.txt"................................................. II
        }
      }
    }
  },
  "mappings": {
    "properties": {
      "code":{
        "type": "keyword"
      },
      "context":{
        "type": "text",
        "analyzer": "my_whitespace"
      },
      "color":{
        "type": "text",
        "analyzer": "my_whitespace"
      }
    }
  }
}
```

* 注：

​		I：`my_whitespace`为自定义分词器

​		II：此处的synonyms_path为es文件夹中以config文件夹为基准的相对路径

### 在相应路径中存入近义词文件

``` txt
Women,women,girl,girls
yellow,orange,wheat
blue,skyblue
white,snow,silver
dress,dresses,skirt,skirts
autumn,fall
shirt,shirts
A,B,C
```

### 存入测试数据

``` json
POST _bulk
{"index" : {"_index" : "fond_goods", "_id":1}}
{"code" : 1,"context" : "ruffled shirt for women 2021 fall slim fit pure color all matching off-neck lantern long sleeve slim women short shirt", "color": "red"}
{"index" : {"_index" : "fond_goods", "_id":2}}
{"code" : 2,"context" : "2021 warmth pullover sweater fall", "color": "blue"}
{"index" : {"_index" : "fond_goods", "_id":3}}
{"code" : 3,"context" : "early autumn elegant dress women dress 2021 autumn new long sleeve", "color": "yellow"}
{"index" : {"_index" : "fond_goods", "_id":4}}
{"code" : 4,"context" : "2021 autumn new  sweater yama autumn and winter female  autumn and winter dot cardigan knitted coat", "color": "snow"}
{"index" : {"_index" : "fond_goods", "_id":5}}
{"code" : 5,"context" : "za satin party dinner skirts suits woemn sexy bandage shirts and high split skirt elegant luxurious female dinner sets", "color": "white"}
{"index" : {"_index" : "fond_goods", "_id":6}}
{"code" : 6,"context" : "big bow tie sweet puff sleeve shirt dress long sleeve shirt skirt solid color shirt dress short skirt ", "color": "moss green"}
{"index" : {"_index" : "fond_goods", "_id":7}}
{"code" : 7,"context" : "casual button plaid short skirts women streetwear a-line summer skirts female high waist yellow autumn short skirts", "color": "skyblue "}
{"index" : {"_index" : "fond_goods", "_id":8}}
{"code" : 8,"context" : "muslim middle east women fashion dress abaya long dress muslim dress arab dress dres", "color": "orange"}
{"index" : {"_index" : "fond_goods", "_id":9}}
{"code" : 9,"context" : "sexy white party dresses autumn winter sexy mini dresses women fashion solid color off shoulder short", "color": "wheat"}
{"index" : {"_index" : "fond_goods", "_id":10}}
{"code" : 10,"context" : "women green patchwork buttons bodycon mini dresses all-match office ladies long shirt dresses autumn party vestidos new", "color": "silver"}
{"index" : {"_index" : "fond_goods_demo", "_id":11}}
{"code" : 11,"context" : "A", "color": "silver"}
{"index" : {"_index" : "fond_goods_demo", "_id":12}}
{"code" : 12,"context" : "B", "color": "silver"}
{"index" : {"_index" : "fond_goods_demo", "_id":13}}
{"code" : 13,"context" : "C", "color": "silver"}
```

### 简单应用

#### 简单尝试一下近义词库查询

* 查询条件

```  json
GET fond_goods/_search
{
  "query": {
    "match": {
      "context": "A"
    }
  }
}
```

* 查询结果

``` json
{
  "took" : 0,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 3,
      "relation" : "eq"
    },
    "max_score" : 2.7354302,
    "hits" : [
      {
        "_index" : "fond_goods",
        "_type" : "_doc",
        "_id" : "11",
        "_score" : 2.7354302,
        "_source" : {
          "code" : 11,
          "context" : "A",
          "color" : "silver"
        }
      },
      {
        "_index" : "fond_goods",
        "_type" : "_doc",
        "_id" : "12",
        "_score" : 2.7354302,
        "_source" : {
          "code" : 12,
          "context" : "B",
          "color" : "silver"
        }
      },
      {
        "_index" : "fond_goods",
        "_type" : "_doc",
        "_id" : "13",
        "_score" : 2.7354302,
        "_source" : {
          "code" : 13,
          "context" : "C",
          "color" : "silver"
        }
      }
    ]
  }
}

```

#### 删除数据

* 删除语句

```json
POST fond_goods/_delete_by_query
{
  "query": {
    "match": {
      "context": "A"
    }
  }
}
```

* 删除结果

``` json
{
  "took" : 5,
  "timed_out" : false,
  "total" : 3,
  "deleted" : 3,
  "batches" : 1,
  "version_conflicts" : 0,
  "noops" : 0,
  "retries" : {
    "bulk" : 0,
    "search" : 0
  },
  "throttled_millis" : 0,
  "requests_per_second" : -1.0,
  "throttled_until_millis" : 0,
  "failures" : [ ]
}
```

**我们一共插入了三条A、B、C这组同义词的数据，一共删除了三条数据；可以看出，在删除时，我们也将A的近义词B、C给删除了**

#### 结论

1. 我们使用A为查询条件，但结果中出现了B、C的数据，即近义词查询成功
2. 我们以A为查询条件，而结果的相关性打分中，B、C的得分与A一致，即表明在查询时，A、B、C是完全等价的，es的相关性打分无法做出区分
3. 在根据条件删除数据时，近义词的数据也会一同删除

### 动态更新近义词文件

es本身提供的近义词功能是在项目启动时读取近义词表文件，并且每一次近义词表文件有更新时都得重启才能再次读取，这就给我们项目使用带来了很大的不便性。

可以使用一款叫做 `elasticsearch-analysis-dynamic-synonym`的es插件来动态读取近义词文件

#### 插件地址

https://github.com/bells/elasticsearch-analysis-dynamic-synonym

#### 插件使用方法

插件使用方法在项目中有详细介绍，这里简单介绍一下

1. 拷贝项目到本地
2. 将项目打包
3. 在es的 `plugins/` 文件夹中新建`dynamic-synonym`文件夹
4. 将` target/releases/elasticsearch-analysis-dynamic-synonym-{version}.zip`文件解压到`dynamic-synonym`中
5. 创建es索引时将同义词配置中的`"type": "synonym"`

``` json
      "filter": {
        "synonymous_filter":{
          "type": "synonym",
          "synonyms_path": "synonym.txt"
        }
      }
```

修改成`"type": "dynamic_synonym"`

```json
      "filter": {
        "synonymous_filter":{
          "type": "dynamic_synonym",
          "synonyms_path": "synonym.txt"
        }
      }
```

**注：该插件还提供了一个可选参数`interval`，即刷新同义词文件时间间隔，默认值为60s**

6. 他与原有操作一致，至此，每隔`60s`，es会自动获取一次同义词文件修改时间，如有变化，es会重新载入同义词文件

### 同义词查询原理

#### 分词

想了解同义词查询的原理就必须先了解es的 分词 （Trem）。ES中的分词（Analysis）就是把一段文本拆分成一系列的单词，也叫做文本分析。在es中，分析器（Analyzer）负责处理这一系列操作。

![分词演示](D:\谭远\typora文件\博客文件夹\elasticsearch\image-20220719133522888.png)

ES的分词器主要由字符过滤器（Character Filter）、分词器（Tokenizer）、分词过滤器（Token Filter）组成。

* 字符过滤器（Character Filter）
  1. 以字符流的形式接受文本，并可以通过添加、删除或更改字符来转化文本。
  2. 一个Analyzer可以由0个或多个字符过滤器
* 分词器（Tokenizer）
  1. 对经过字符过滤器过滤后的文本按照一定规则分词。一个Analyzer只允许有一个分词器
* 分词过滤器（Token Filter）
  1. 针对分词后的token再次进行过滤，可以增删和修改token，一个分词器中可以有多个token过滤器

#### 同义词过滤器

同义词查询的关键其实就是自定义Token过滤器。该过滤器在收到分词器发过来的数据（我暂时将其称之为分词数据）时，会先读取用户存放的近义词文件，比对分词数据。当出现同义词时，Token过滤器就按照近义词文件配置的规则选定带搜索词组，进行同义词搜索。

我们可以拿之前的索引做个试验：我们的索引使用的是自定义的分析器`my_whitespace`，其中分词器是`whitespace`空格分词器， 而token Filter 使用的是自定义的近义词过滤器。由上述可知，我们自定义的分析器与官方自带的`whitespace`分析器唯一的差别就在token Filter上。

##### 我们使用官方的`whitespace`分析器来看一下分词情况：

```json
GET fond_goods/_analyze
{
  "analyzer": "whitespace",
  "field":"context", 
  "text": "A"
}
```

* 结果

``` json
{
  "tokens" : [
    {
      "token" : "A",
      "start_offset" : 0,
      "end_offset" : 1,
      "type" : "word",
      "position" : 0
    }
  ]
}
```

在经过分析器后，字符A被分成了 "A"这一个分词

* 再来尝试一个长度更长的字符串

``` json
GET fond_goods/_analyze
{
  "analyzer": "whitespace",
  "field":"context", 
  "text": "ruffled shirt for women 2021 fall slim fit pure color all matching off-neck lantern long sleeve slim women short shirt"
}

```

* 结果

``` json
{
  "tokens" : [
    {
      "token" : "ruffled",
      "start_offset" : 0,
      "end_offset" : 7,
      "type" : "word",
      "position" : 0
    },
    {
      "token" : "shirt",
      "start_offset" : 8,
      "end_offset" : 13,
      "type" : "word",
      "position" : 1
    },
    {
      "token" : "for",
      "start_offset" : 14,
      "end_offset" : 17,
      "type" : "word",
      "position" : 2
    },
    {
      "token" : "women",
      "start_offset" : 18,
      "end_offset" : 23,
      "type" : "word",
      "position" : 3
    },
    {
      "token" : "2021",
      "start_offset" : 24,
      "end_offset" : 28,
      "type" : "word",
      "position" : 4
    },
    {
      "token" : "fall",
      "start_offset" : 29,
      "end_offset" : 33,
      "type" : "word",
      "position" : 5
    },
    {
      "token" : "slim",
      "start_offset" : 34,
      "end_offset" : 38,
      "type" : "word",
      "position" : 6
    },
    {
      "token" : "fit",
      "start_offset" : 39,
      "end_offset" : 42,
      "type" : "word",
      "position" : 7
    },
    {
      "token" : "pure",
      "start_offset" : 43,
      "end_offset" : 47,
      "type" : "word",
      "position" : 8
    },
    {
      "token" : "color",
      "start_offset" : 48,
      "end_offset" : 53,
      "type" : "word",
      "position" : 9
    },
    {
      "token" : "all",
      "start_offset" : 54,
      "end_offset" : 57,
      "type" : "word",
      "position" : 10
    },
    {
      "token" : "matching",
      "start_offset" : 58,
      "end_offset" : 66,
      "type" : "word",
      "position" : 11
    },
    {
      "token" : "off-neck",
      "start_offset" : 67,
      "end_offset" : 75,
      "type" : "word",
      "position" : 12
    },
    {
      "token" : "lantern",
      "start_offset" : 76,
      "end_offset" : 83,
      "type" : "word",
      "position" : 13
    },
    {
      "token" : "long",
      "start_offset" : 84,
      "end_offset" : 88,
      "type" : "word",
      "position" : 14
    },
    {
      "token" : "sleeve",
      "start_offset" : 89,
      "end_offset" : 95,
      "type" : "word",
      "position" : 15
    },
    {
      "token" : "slim",
      "start_offset" : 96,
      "end_offset" : 100,
      "type" : "word",
      "position" : 16
    },
    {
      "token" : "women",
      "start_offset" : 101,
      "end_offset" : 106,
      "type" : "word",
      "position" : 17
    },
    {
      "token" : "short",
      "start_offset" : 107,
      "end_offset" : 112,
      "type" : "word",
      "position" : 18
    },
    {
      "token" : "shirt",
      "start_offset" : 113,
      "end_offset" : 118,
      "type" : "word",
      "position" : 19
    }
  ]
}

```

* 结果

可以看到，`whitespace`分析器将输入字符串按照空格拆分成了如上结果

##### 我们再来试试自定义的分析器

``` json
GET fond_goods/_analyze
{
  "analyzer": "my_whitespace",
  "field":"context", 
  "text": "A"
}
```

* 结果

``` json
{
  "tokens" : [
    {
      "token" : "A",
      "start_offset" : 0,
      "end_offset" : 1,
      "type" : "word",
      "position" : 0
    },
    {
      "token" : "B",
      "start_offset" : 0,
      "end_offset" : 1,
      "type" : "SYNONYM",
      "position" : 0
    },
    {
      "token" : "C",
      "start_offset" : 0,
      "end_offset" : 1,
      "type" : "SYNONYM",
      "position" : 0
    }
  ]
}
```

经过分析器后，A这个字符被分成了 A、B、C三个分词，且在`type`字段上有作区分，A被标记为`word`，B、C被标记为` SYNONYM` 

* 我们再尝试一下长字符串（注：在近义词文件中，我们定义了shirt,shirts为一组近义词；Women,women,girl,girls为一组近义词）

``` json
GET fond_goods/_analyze
{
  "analyzer": "my_whitespace",
  "field":"context", 
  "text": "ruffled shirt for women 2021 fall slim fit pure color all matching off-neck lantern long sleeve slim women short shirt"
}
```

* 结果

``` json
{
  "tokens" : [
    {
      "token" : "ruffled",
      "start_offset" : 0,
      "end_offset" : 7,
      "type" : "word",
      "position" : 0
    },
    {
      "token" : "shirt",
      "start_offset" : 8,
      "end_offset" : 13,
      "type" : "word",
      "position" : 1
    },
    {
      "token" : "shirts",
      "start_offset" : 8,
      "end_offset" : 13,
      "type" : "SYNONYM",
      "position" : 1
    },
    {
      "token" : "for",
      "start_offset" : 14,
      "end_offset" : 17,
      "type" : "word",
      "position" : 2
    },
    {
      "token" : "women",
      "start_offset" : 18,
      "end_offset" : 23,
      "type" : "word",
      "position" : 3
    },
    {
      "token" : "Women",
      "start_offset" : 18,
      "end_offset" : 23,
      "type" : "SYNONYM",
      "position" : 3
    },
    {
      "token" : "girl",
      "start_offset" : 18,
      "end_offset" : 23,
      "type" : "SYNONYM",
      "position" : 3
    },
    {
      "token" : "girls",
      "start_offset" : 18,
      "end_offset" : 23,
      "type" : "SYNONYM",
      "position" : 3
    },
    {
      "token" : "2021",
      "start_offset" : 24,
      "end_offset" : 28,
      "type" : "word",
      "position" : 4
    },
    {
      "token" : "fall",
      "start_offset" : 29,
      "end_offset" : 33,
      "type" : "word",
      "position" : 5
    },
    {
      "token" : "autumn",
      "start_offset" : 29,
      "end_offset" : 33,
      "type" : "SYNONYM",
      "position" : 5
    },
    {
      "token" : "slim",
      "start_offset" : 34,
      "end_offset" : 38,
      "type" : "word",
      "position" : 6
    },
    {
      "token" : "fit",
      "start_offset" : 39,
      "end_offset" : 42,
      "type" : "word",
      "position" : 7
    },
    {
      "token" : "pure",
      "start_offset" : 43,
      "end_offset" : 47,
      "type" : "word",
      "position" : 8
    },
    {
      "token" : "color",
      "start_offset" : 48,
      "end_offset" : 53,
      "type" : "word",
      "position" : 9
    },
    {
      "token" : "all",
      "start_offset" : 54,
      "end_offset" : 57,
      "type" : "word",
      "position" : 10
    },
    {
      "token" : "matching",
      "start_offset" : 58,
      "end_offset" : 66,
      "type" : "word",
      "position" : 11
    },
    {
      "token" : "off-neck",
      "start_offset" : 67,
      "end_offset" : 75,
      "type" : "word",
      "position" : 12
    },
    {
      "token" : "lantern",
      "start_offset" : 76,
      "end_offset" : 83,
      "type" : "word",
      "position" : 13
    },
    {
      "token" : "long",
      "start_offset" : 84,
      "end_offset" : 88,
      "type" : "word",
      "position" : 14
    },
    {
      "token" : "sleeve",
      "start_offset" : 89,
      "end_offset" : 95,
      "type" : "word",
      "position" : 15
    },
    {
      "token" : "slim",
      "start_offset" : 96,
      "end_offset" : 100,
      "type" : "word",
      "position" : 16
    },
    {
      "token" : "women",
      "start_offset" : 101,
      "end_offset" : 106,
      "type" : "word",
      "position" : 17
    },
    {
      "token" : "Women",
      "start_offset" : 101,
      "end_offset" : 106,
      "type" : "SYNONYM",
      "position" : 17
    },
    {
      "token" : "girl",
      "start_offset" : 101,
      "end_offset" : 106,
      "type" : "SYNONYM",
      "position" : 17
    },
    {
      "token" : "girls",
      "start_offset" : 101,
      "end_offset" : 106,
      "type" : "SYNONYM",
      "position" : 17
    },
    {
      "token" : "short",
      "start_offset" : 107,
      "end_offset" : 112,
      "type" : "word",
      "position" : 18
    },
    {
      "token" : "shirt",
      "start_offset" : 113,
      "end_offset" : 118,
      "type" : "word",
      "position" : 19
    },
    {
      "token" : "shirts",
      "start_offset" : 113,
      "end_offset" : 118,
      "type" : "SYNONYM",
      "position" : 19
    }
  ]
}
```

可以看到，shirt、women两个字符串经过分析器后被分词为了`shirt, shirts`以及 `women, Women, girl, girls`两组分词，且都做了相应标识。

## 参考文章

同义词搜索原理部分参考

https://blog.csdn.net/woshixubo123/article/details/121774972 

以及

https://blog.csdn.net/woshixubo123/article/details/121898514

两篇文章

其他均来自于官网或者自己举的例子
