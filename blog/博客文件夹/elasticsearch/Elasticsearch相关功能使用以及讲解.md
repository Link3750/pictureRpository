# Elasticsearch相关功能使用以及讲解

## painless脚本语言

painless是一门专为es编写、调优的脚本语言，使用者能使用该语言安全编写脚本，将其插入到ES语句中，也能够存放在任何ES支持的地方。

painless在满足以下原则下提供了丰富的其他功能：

1. 安全性：为了保证ES集群的安全性，painless使用细颗粒度的白名单列表来控制脚本运行。任何不在列表上的结果都将抛出异常。完整的白名单详见 [PainliessAPI Reference](https://www.elastic.co/guide/en/elasticsearch/painless/master/painless-api-reference.html)。
2. 高效性：painless将直接编译进JVM字节码中，这是JVM所能够提供的最优优化方案。同时，painless在运行时会避免任何减少效率的额外检查。
3. 简洁性：painless的语法同市面上大多基础语言的语法类似。painless语法包含java语法的子集，以及一些其他的补充来增强可读性。

## 如何编写脚本

在Elasticsearch API任何支持编写脚本的地方都遵循以下的脚本编写语法：

``` json
"script": {
    'lang': "<specify language>",
    "source" | "id"： "your inline script contents or stored script id.",
    "params": {"the parameters injected in your script contents"}
}
```

* lang：你需要使用的脚本语言，默认为painliess；
* source，id：在source后填写你的脚本内容，在id后面填写存储好的脚本文件的id；
* params：脚本语言中所使用的参数变量。通过该种方式别写的脚本效率要好于直接将变量在脚本内赋值的硬编码方式。

### *注：以上所有内容大部分由笔者翻译自官方文档，翻译有所删减，也可能有误，请自行甄别*

## 评分脚本

拿评分脚本来举例：

由于我们的搜索业务需求中需要对搜索出来的商品进行自定义排序，而es原本的打分无法完全满足需求，因此要对es的打分规则进行修改，来满足业务需求。

*注：如果想了解es打分规则可参考另一篇文档，想详细了解如何修改es原本的打分规则，也请参看另一篇文档*

一条普通的搜索语句：

``` json
GET fond_publish/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "match": {
            "context": "denim dress"
          }
        }
      ]
    }
  }
}
```

出来的结果：

``` json
{
  "took" : 33,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 10000,
      "relation" : "gte"
    },
    "max_score" : 6.0628376,
    "hits" : [
      {
        "_index" : "fond_publish",
        "_type" : "_doc",
        "_id" : "1365187704263036929",
        "_score" : 6.0628376,
        "_source" : {
          "code" : "1365187704263036929",
          "skuScore" : 100.0,
          "context" : "dress  ripped denim casual dress stretch denim dress f00010605 black,navy blue cmz"
        }
      },
      {
        "_index" : "fond_publish",
        "_type" : "_doc",
        "_id" : "1442725377638453249",
        "_score" : 5.9616523,
        "_source" : {
          "code" : "1442725377638453249",
          "skuScore" : 107.0,
          "context" : "denim jacket women  autumn and winter new slim denim dress fashion europe denim women  dress sexy f00075073 light blue ozxx"
        }
      },
      {
        "_index" : "fond_publish",
        "_type" : "_doc",
        "_id" : "1384812109196730369",
        "_score" : 5.906888,
        "_source" : {
          "code" : "1384812109196730369",
          "skuScore" : 106.0,
          "context" : "denim dress  new short sleeve blouse collar denim dress f00019847 gray,light blue,navy blue,bright blue ccam"
        }
      },
      {
        "_index" : "fond_publish",
        "_type" : "_doc",
        "_id" : "1409318809253539841",
        "_score" : 5.874957,
        "_source" : {
          "code" : "1409318809253539841",
          "consumerCode" : "public",
          "context" : "2021 sexy tight women denim dress denim f00033786 navy blue,light blue yupeng"
        }
      },
      {
        "_index" : "fond_publish",
        "_type" : "_doc",
        "_id" : "1456494147439714305",
        "_score" : 5.874957,
        "_source" : {
          "code" : "1456494147439714305",
          "skuScore" : 209.0,
          "context" : "new denim vest coat suit collar denim dress long f00087360 blue caijun"
        }
      },
      {
        "_index" : "fond_publish",
        "_type" : "_doc",
        "_id" : "1456511193858727937",
        "_score" : 5.8729877,
        "_source" : {
          "code" : "1456511193858727937",
          "skuScore" : 109.0,
          "context" : "popular 2021 summer ruffles skirt stretch denim dress women  clothing f00087519 light blue,blue-ruffled denim dress colore"
        }
      },
      {
        "_index" : "fond_publish",
        "_type" : "_doc",
        "_id" : "1421291612122931201",
        "_score" : 5.8345222,
        "_source" : {
          "code" : "1421291612122931201",
          "skuScore" : 107.0,
          "context" : "denim long skirts dress skirt denim spring summer slim-fit slimming f00055258 blue stylebobon"
        }
      },
      {
        "_index" : "fond_publish",
        "_type" : "_doc",
        "_id" : "1248879746580893697",
        "_score" : 5.810824,
        "_source" : {
          "code" : "1248879746580893697",
          "skuScore" : 100.0,
          "context" : "sexy long denim dress with belt vintage button front denim dress spring autumn slim ladies office dress s20dr6511 blue simplee"
        }
      },
      {
        "_index" : "fond_publish",
        "_type" : "_doc",
        "_id" : "1408272828265259009",
        "_score" : 5.7946835,
        "_source" : {
          "code" : "1408272828265259009",
          "skuScore" : 6.0,
          "context" : "2021 stretch fit denim dress shirt sleeve denim long skirts f00033550 light blue yupeng",
          "englishName" : "Dresses"
        }
      },
      {
        "_index" : "fond_publish",
        "_type" : "_doc",
        "_id" : "1446657562930139137",
        "_score" : 5.7946835,
        "_source" : {
          "code" : "1446657562930139137",
          "skuScore" : 214.0,
          "context" : "fashion sexy v neck denim stitching irregular dress  plus size f00076758 denim color coya"
        }
      }
    ]
  }
}
```

可以看出其打分是es自带的一个打分系统进行打分的，而这个打分结果而成的排序结果与我们的业务需求不符，因此需要使用打分脚本来控制打分情况，从而改变商品排序状态。

利用打分脚本进行打分：

``` json
GET fond_publish/_search
{
  "query": {
    "function_score": {
      "query": {
        "bool": {
          "must": [
            {
              "match": {
                "context": "denim dress"
              }
            }
          ]
        }
      },
     "functions" : [
       {
         "script_score" : {
           "script" : {
            "source": """
          if(doc['skuScore'].size() != 0) {
            return  doc["skuScore"].value + params.relevanceRateScore * 1000 * _score;
          }
          """,
             "lang" : "painless",
             "params" : {
               "relevanceRateScore" : 0.1
             }
           }
         }
       }
     ],
     "boost_mode" : "replace",
     "max_boost" : 3.4028235E38,
     "boost" : 1
    }
  }
}
```

结果：

``` json
{
  "took" : 11,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 10000,
      "relation" : "gte"
    },
    "max_score" : 845.69904,
    "hits" : [
      {
        "_index" : "fond_publish",
        "_type" : "_doc",
        "_id" : "1421127793383858178",
        "_score" : 845.69904,
        "_source" : {
          "code" : "1421127793383858178",
          "context" : "2021 autumn new women loose casual puff sleeve denim dress f00055194 blue fsypc",
          "skuScore" : 307.0
        }
      },
      {
        "_index" : "fond_publish",
        "_type" : "_doc",
        "_id" : "1425726206545010690",
        "_score" : 840.0865,
        "_source" : {
          "code" : "1425726206545010690",
          "context" : "women  clothing nightclub uniforms denim pleated gradient stitching single button dress f00062457 pink,blue aselin",
          "skuScore" : 307.0
        }
      },
      {
        "_index" : "fond_publish",
        "_type" : "_doc",
        "_id" : "1420216365126893569",
        "_score" : 839.0865,
        "_source" : {
          "code" : "1420216365126893569",
          "context" : "2021 new  off-shoulder skinny sheath washed sexy denim dress plus size f00052930 blue ofashion",
          "skuScore" : 306.0
        }
      },
      {
        "_index" : "fond_publish",
        "_type" : "_doc",
        "_id" : "1420216649764945921",
        "_score" : 839.0865,
        "_source" : {
          "code" : "1420216649764945921",
          "context" : "new lace stitching ripped tight denim sexy dress  new women clothes f00052937 blue ofashion",
          "skuScore" : 306.0
        }
      },
      {
        "_index" : "fond_publish",
        "_type" : "_doc",
        "_id" : "1456494147439714305",
        "_score" : 796.4957,
        "_source" : {
          "code" : "1456494147439714305",
          "context" : "new denim vest coat suit collar denim dress long f00087360 blue caijun",
          "skuScore" : 209.0
        }
      },
      {
        "_index" : "fond_publish",
        "_type" : "_doc",
        "_id" : "1446657562930139137",
        "_score" : 793.4683,
        "_source" : {
          "code" : "1446657562930139137",
          "context" : "fashion sexy v neck denim stitching irregular dress  plus size f00076758 denim color coya",
          "skuScore" : 214.0
        }
      },
      {
        "_index" : "fond_publish",
        "_type" : "_doc",
        "_id" : "1426410735089786881",
        "_score" : 784.6739,
        "_source" : {
          "code" : "1426410735089786881",
          "context" : "new women clothing 2021 new blue color denim off-the-shoulder flared sleeves denim dress f00063617 blue bnfs",
          "skuScore" : 213.0
        }
      },
      {
        "_index" : "fond_publish",
        "_type" : "_doc",
        "_id" : "1425384661966954497",
        "_score" : 781.74475,
        "_source" : {
          "code" : "1425384661966954497",
          "context" : "new denim dress short sleeve slim dress f00061755 blue caijun",
          "skuScore" : 213.0
        }
      },
      {
        "_index" : "fond_publish",
        "_type" : "_doc",
        "_id" : "1425267248084590593",
        "_score" : 780.8443,
        "_source" : {
          "code" : "1425267248084590593",
          "context" : "fashion printed sexy fitted waist figure flattering shirt maxi dress f00061002 khaki letter,white,white square,striped tie-dye,coral red,blue letters,black stripe,yellow leaves,red tie dye,khaki,black yellow flower,imitation denim tie-dye xxbjl",
          "skuScore" : 307.0
        }
      },
      {
        "_index" : "fond_publish",
        "_type" : "_doc",
        "_id" : "1418494316197036033",
        "_score" : 772.511,
        "_source" : {
          "code" : "1418494316197036033",
          "context" : "new ripped denim shorts loose straight-leg denim shorts f00052066 light-colored caijun",
          "skuScore" : 313.0
        }
      }
    ]
  }
}
```

#### 注意事项：

1. 脚本里面返回的结果不允许为空，否则会报错

#### 参数讲解

1. boost_mode：脚本计算结果与原打分结果的结合方式，参数有：
   1. replace：替换
   2. sum：相加
   3. multiply：相乘
   4. min：二者取最小
   5. max：二者取最大
   6. avg：取平均值
2. max_boost：计算结果的最大值
   1. *注：此计算结果最大值为脚本结果的最大值，并不会影响es本身的打分规则*
   2. 结果溢出的将直接丢弃，而不是整个结果同比例缩小
3. boost：结果的影响因子，该数越大，计算结果越大

## 对搜索结果进行重打分

由于我们业务逻辑中搜索词含有多个时允许半匹配，而业务需求又要把全匹配的结果排名更高，因此需要设计此方法来对搜索结果进行重打分，以满足业务需求。

es中提供了Rescore方法来对结果进行重打分

``` json
GET fond_publish/_search
{
  "query": {
    "function_score": {
      "query": {
        "bool": {
          "must": [
            {
              "match": {
                "context": "denim dress"
              }
            }
          ]
        }
      },
     "functions" : [
       {
         "script_score" : {
           "script" : {
            "source": """
          if(doc['skuScore'].size() != 0) {
            return  doc["skuScore"].value + params.relevanceRateScore * 1000 * _score;
          }
          """,
             "lang" : "painless",
             "params" : {
               "relevanceRateScore" : 0.1
             }
           }
         }
       }
     ],
     "boost_mode" : "replace",
     "max_boost" : 3.4028235E38,
     "boost" : 1
    }
  },
  "rescore": {
    "query": {
      "rescore_query":{
        "bool":{
          "must":{
            "match":{
          "context" : {
            "query" : "denim dress",
            "operator" : "and",
            "boost": 1.5
          }
        }
          }
        }
      },
    "query_weight": 1,
    "rescore_query_weight":1
    },
    "window_size": 50
  }
}
```

搜索结果：

``` json
{
  "took" : 74,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 10000,
      "relation" : "gte"
    },
    "max_score" : 853.77954,
    "hits" : [
      {
        "_index" : "fond_publish",
        "_type" : "_doc",
        "_id" : "1421127793383858178",
        "_score" : 853.77954,
        "_source" : {
          "code" : "1421127793383858178",
          "context" : "2021 autumn new women loose casual puff sleeve denim dress f00055194 blue fsypc",
          "skuScore" : 307.0
        }
      },
      {
        "_index" : "fond_publish",
        "_type" : "_doc",
        "_id" : "1425726206545010690",
        "_score" : 848.08276,
        "_source" : {
          "code" : "1425726206545010690",
          "context" : "women  clothing nightclub uniforms denim pleated gradient stitching single button dress f00062457 pink,blue aselin",
          "skuScore" : 307.0
        }
      },
      {
        "_index" : "fond_publish",
        "_type" : "_doc",
        "_id" : "1420216365126893569",
        "_score" : 847.08276,
        "_source" : {
          "code" : "1420216365126893569",
          "context" : "2021 new  off-shoulder skinny sheath washed sexy denim dress plus size f00052930 blue ofashion",
          "skuScore" : 306.0
        }
      },
      {
        "_index" : "fond_publish",
        "_type" : "_doc",
        "_id" : "1420216649764945921",
        "_score" : 847.08276,
        "_source" : {
          "code" : "1420216649764945921",
          "consumerCode" : "public",
          "context" : "new lace stitching ripped tight denim sexy dress  new women clothes f00052937 blue ofashion",
          "skuScore" : 306.0
        }
      },
      {
        "_index" : "fond_publish",
        "_type" : "_doc",
        "_id" : "1456494147439714305",
        "_score" : 805.30817,
        "_source" : {
          "code" : "1456494147439714305",
          "context" : "new denim vest coat suit collar denim dress long f00087360 blue caijun",
          "skuScore" : 209.0
        }
      },
      {
        "_index" : "fond_publish",
        "_type" : "_doc",
        "_id" : "1446657562930139137",
        "_score" : 802.16034,
        "_source" : {
          "code" : "1446657562930139137",
          "context" : "fashion sexy v neck denim stitching irregular dress  plus size f00076758 denim color coya",
          "skuScore" : 214.0
        }
      },
      {
        "_index" : "fond_publish",
        "_type" : "_doc",
        "_id" : "1426410735089786881",
        "_score" : 793.249,
        "_source" : {
          "code" : "1426410735089786881",
          "context" : "new women clothing 2021 new blue color denim off-the-shoulder flared sleeves denim dress f00063617 blue bnfs",
          "skuScore" : 213.0
        }
      },
      {
        "_index" : "fond_publish",
        "_type" : "_doc",
        "_id" : "1425384661966954497",
        "_score" : 790.27594,
        "_source" : {
          "code" : "1425384661966954497",
          "context" : "new denim dress short sleeve slim dress f00061755 blue caijun","skuScore" : 213.0
        }
      },
      {
        "_index" : "fond_publish",
        "_type" : "_doc",
        "_id" : "1425267248084590593",
        "_score" : 787.95197,
        "_source" : {
          "code" : "1425267248084590593",
          "context" : "fashion printed sexy fitted waist figure flattering shirt maxi dress f00061002 khaki letter,white,white square,striped tie-dye,coral red,blue letters,black stripe,yellow leaves,red tie dye,khaki,black yellow flower,imitation denim tie-dye xxbjl",
          "skuScore" : 307.0
        }
      },
      {
        "_index" : "fond_publish",
        "_type" : "_doc",
        "_id" : "1418494316197036033",
        "_score" : 772.511,
        "_source" : {
          "code" : "1418494316197036033",
          "context" : "new ripped denim shorts loose straight-leg denim shorts f00052066 light-colored caijun",
          "skuScore" : 313.0
        }
      }
    ]
  }
}
```

`window_size` 窗口大小，默认值是from和size参数值之和，它指定了每个分片上参与二次评分的文档个数
`query_weight` 查询权重，默认值是1，原始查询得分与二次评分的得分相加之前将乘以改值
`rescore_query_weight` 二次评分查询的权重值，默认值是1，二次评分查询得分在与原始查询得分相加之前，乘以该值

















