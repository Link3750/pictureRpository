# ES 打分机制

## ES搜索流程

![MatchQuery流程](https://raw.githubusercontent.com/Link3750/pictureRpository/main/pic/202301032041269.png)

ES会对搜索出来的结果进行一个打分，然后根据这个打分结果来默认排序。了解这种打分机制对于使用es来存取信息的业务来说便是十分必要的。

## ES的两个评分模型

1. TF/IDF
2. BM25

两种方法虽然虽然属于不同的模型，但他们的评分公式差别并不大，都使用了IDF和TD算法来定义某个词的权重，然后把查询匹配的词的权重相加作为整片文档的分数。

例如：我查询denim dress这个词组，es会把这个词组拆分成`denim`和`dress`两个单词来分别进行查询、打分，最后将打分的结果相加起来即为`denim dress` 这个词组搜索结果的最终得分。

## 相关性算法

###  基础概念

相关性算法分为以下几个部分：

1. 词频（Term Frequency：TF）：即单词在该文档中出现的次数，词频越高，相关度越高
2. 文档频率（Document Frequency：DF）：即单词出现的文档数
3. 逆向文档频率（Inverse Document Frequency：IDF）：与DF相反，可以简单理解为其倒数。即单词出现的文档数越少，相关性越高。
4. Field-length Norm：文档越短，相关性越高，文档长度越长，相关性越弱。

### 太难不看版：

* 所谓的TF/IDF模型简单来说就可以理解为TF、IDF、以及Field-length Norm以及一些其他算法结果的和即为最后的总得分。
* 而BM25模型则是在TD/IDF的基础上进行了一定程度的优化，避免了在某些情况下TF/IDF的计算曲线斜率过陡。

### 太难还坚持看版：

*不好意思，没来得及写，以后再补*

#### TF/IDF公式：

$$
score(q,d) = queryNorm(q) * coord(q,d) * SUM(tf(t in d), idf(t)^2, t.getBoost(), norm(t,d))(t-in-q)
$$

其中：

* score(q,d)：搜索条件q在文档d中的相关性得分
* queryNorm(q)：标准化因子
* coord(q, d)：协调因子
* SUM(tf(t in d), idf(t)^2, t.getBoost(), norm(t,d))：在搜索条件q中的分词t在各项各项公式中计算结果的和
  * tf(t in d)：分词t在文档d中出现的频率
  *  idf(t)：分词t在所有文档中出现频率的“逆数”
  * t.getBoost()：分词t所占的比重
  * norm(t,d)：搜索字段的长度

## 实际操作观察、改变ES的打分情况

### 观察es打分细节

es的api提供了`explain:true`来展示内部打分细节：

```json
GET demo_index/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "match": {
            "demoName": {
              "query": "测试", 
              "fuzziness": "auto"
            }
          }
        }
      ]
    }
  },
  "explain": true
}
```



### 改变es打分

由于es5版本以后打分机制都默认使用的BM25模型来计算，而在创建索引时，es提供了方法改变某些参数，从而影响es的打分情况。

创建索引时往settings中写如下脚本：

``` json
"similarity":{
      "my_bm25":{
        "type":"BM25",
        "b":0.75,
        "k1":0.3
      }
 }
```

#### 参数解释

- **k1**：控制对于得分而言词频（TF）的重要性，默认为1.2。
- **b**：是介于0 ~ 1之间的数值，控制文档篇幅对于得分的影响程度，默认为0.75。

而在创建字段类型的时候将以下脚本写入待修改的参数中：

``` json
  "mappings":{
    "doc":{
      "properties":{
        "context":{
          "type":"keyword",
          "similarity":"my_bm25"
        }
      }
    }
  }
```

如果我们要使用某种特定的打分模型，并且希望应用到全局，那么就在elasticsearch.yml配置文件中加入：

``` yml
index.similarity.default.type: BM25
```

### 修改参数后的得分详细情况

``` json
{
        "_shard" : "[fond_publish][0]",
        "_node" : "8BYlD1WSTfiyJ6XBmqjvKg",
        "_index" : "fond_publish",
        "_type" : "_doc",
        "_id" : "1421127793383858178",
        "_score" : 853.77954,
        "_source" : {
          "code" : "1421127793383858178",
          "context" : "2021 autumn new women loose casual puff sleeve denim dress f00055194 blue fsypc",
          "skuScore" : 307.0,
        },
        "_explanation" : {
          "value" : 853.77954,
          "description" : "sum of:",
          "details" : [
            {
              "value" : 845.69904,
              "description" : "product of:",
              "details" : [
                {
                  "value" : 845.69904,
                  "description" : "min of:",
                  "details" : [
                    {
                      "value" : 845.69904,
                      "description" : "script score function, computed with script:\"Script{type=inline, lang='painless', idOrCode='\n          if(doc['skuScore'].size() != 0) {\n            return  doc[\"skuScore\"].value + params.relevanceRateScore * 1000 * _score;\n          }\n          ', options={}, params={relevanceRateScore=0.1}}\"",
                      "details" : [
                        {
                          "value" : 5.3869905,
                          "description" : "_score: ",
                          "details" : [
                            {
                              "value" : 5.3869905,
                              "description" : "sum of:",
                              "details" : [
                                {
                                  "value" : 4.0939107,
                                  "description" : "weight(context:denim in 1271) [PerFieldSimilarity], result of:",
                                  "details" : [
                                    {
                                      "value" : 4.0939107,
                                      "description" : "score(freq=1.0), computed as boost * idf * tf from:",
                                      "details" : [
                                        {
                                          "value" : 1.3,
                                          "description" : "boost",
                                          "details" : [ ]
                                        },
                                        {
                                          "value" : 3.9221752,
                                          "description" : "idf, computed as log(1 + (N - n + 0.5) / (n + 0.5)) from:",
                                          "details" : [
                                            {
                                              "value" : 1936,
                                              "description" : "n, number of documents containing term",
                                              "details" : [ ]
                                            },
                                            {
                                              "value" : 97812,
                                              "description" : "N, total number of documents with field",
                                              "details" : [ ]
                                            }
                                          ]
                                        },
                                        {
                                          "value" : 0.80291224,
                                          "description" : "tf, computed as freq / (freq + k1 * (1 - b + b * dl / avgdl)) from:",
                                          "details" : [
                                            {
                                              "value" : 1.0,
                                              "description" : "freq, occurrences of term within document",
                                              "details" : [ ]
                                            },
                                            {
                                              "value" : 0.3,
                                              "description" : "k1, term saturation parameter",
                                              "details" : [ ]
                                            },
                                            {
                                              "value" : 0.75,
                                              "description" : "b, length normalization parameter",
                                              "details" : [ ]
                                            },
                                            {
                                              "value" : 13.0,
                                              "description" : "dl, length of field",
                                              "details" : [ ]
                                            },
                                            {
                                              "value" : 17.158834,
                                              "description" : "avgdl, average length of field",
                                              "details" : [ ]
                                            }
                                          ]
                                        }
                                      ]
                                    }
                                  ]
                                },
                                {
                                  "value" : 1.2930797,
                                  "description" : "weight(context:dress in 1271) [PerFieldSimilarity], result of:",
                                  "details" : [
                                    {
                                      "value" : 1.2930797,
                                      "description" : "score(freq=1.0), computed as boost * idf * tf from:",
                                      "details" : [
                                        {
                                          "value" : 1.3,
                                          "description" : "boost",
                                          "details" : [ ]
                                        },
                                        {
                                          "value" : 1.2388362,
                                          "description" : "idf, computed as log(1 + (N - n + 0.5) / (n + 0.5)) from:",
                                          "details" : [
                                            {
                                              "value" : 28338,
                                              "description" : "n, number of documents containing term",
                                              "details" : [ ]
                                            },
                                            {
                                              "value" : 97812,
                                              "description" : "N, total number of documents with field",
                                              "details" : [ ]
                                            }
                                          ]
                                        },
                                        {
                                          "value" : 0.80291224,
                                          "description" : "tf, computed as freq / (freq + k1 * (1 - b + b * dl / avgdl)) from:",
                                          "details" : [
                                            {
                                              "value" : 1.0,
                                              "description" : "freq, occurrences of term within document",
                                              "details" : [ ]
                                            },
                                            {
                                              "value" : 0.3,
                                              "description" : "k1, term saturation parameter",
                                              "details" : [ ]
                                            },
                                            {
                                              "value" : 0.75,
                                              "description" : "b, length normalization parameter",
                                              "details" : [ ]
                                            },
                                            {
                                              "value" : 13.0,
                                              "description" : "dl, length of field",
                                              "details" : [ ]
                                            },
                                            {
                                              "value" : 17.158834,
                                              "description" : "avgdl, average length of field",
                                              "details" : [ ]
                                            }
                                          ]
                                        }
                                      ]
                                    }
                                  ]
                                }
                              ]
                            }
                          ]
                        }
                      ]
                    },
                    {
                      "value" : 3.4028235E38,
                      "description" : "maxBoost",
                      "details" : [ ]
                    }
                  ]
                },
                {
                  "value" : 1.0,
                  "description" : "primaryWeight",
                  "details" : [ ]
                }
              ]
            },
            {
              "value" : 8.080486,
              "description" : "product of:",
              "details" : [
                {
                  "value" : 8.080486,
                  "description" : "sum of:",
                  "details" : [
                    {
                      "value" : 6.1408668,
                      "description" : "weight(context:denim in 1271) [PerFieldSimilarity], result of:",
                      "details" : [
                        {
                          "value" : 6.1408668,
                          "description" : "score(freq=1.0), computed as boost * idf * tf from:",
                          "details" : [
                            {
                              "value" : 1.9499999,
                              "description" : "boost",
                              "details" : [ ]
                            },
                            {
                              "value" : 3.9221752,
                              "description" : "idf, computed as log(1 + (N - n + 0.5) / (n + 0.5)) from:",
                              "details" : [
                                {
                                  "value" : 1936,
                                  "description" : "n, number of documents containing term",
                                  "details" : [ ]
                                },
                                {
                                  "value" : 97812,
                                  "description" : "N, total number of documents with field",
                                  "details" : [ ]
                                }
                              ]
                            },
                            {
                              "value" : 0.80291224,
                              "description" : "tf, computed as freq / (freq + k1 * (1 - b + b * dl / avgdl)) from:",
                              "details" : [
                                {
                                  "value" : 1.0,
                                  "description" : "freq, occurrences of term within document",
                                  "details" : [ ]
                                },
                                {
                                  "value" : 0.3,
                                  "description" : "k1, term saturation parameter",
                                  "details" : [ ]
                                },
                                {
                                  "value" : 0.75,
                                  "description" : "b, length normalization parameter",
                                  "details" : [ ]
                                },
                                {
                                  "value" : 13.0,
                                  "description" : "dl, length of field",
                                  "details" : [ ]
                                },
                                {
                                  "value" : 17.158834,
                                  "description" : "avgdl, average length of field",
                                  "details" : [ ]
                                }
                              ]
                            }
                          ]
                        }
                      ]
                    },
                    {
                      "value" : 1.9396195,
                      "description" : "weight(context:dress in 1271) [PerFieldSimilarity], result of:",
                      "details" : [
                        {
                          "value" : 1.9396195,
                          "description" : "score(freq=1.0), computed as boost * idf * tf from:",
                          "details" : [
                            {
                              "value" : 1.9499999,
                              "description" : "boost",
                              "details" : [ ]
                            },
                            {
                              "value" : 1.2388362,
                              "description" : "idf, computed as log(1 + (N - n + 0.5) / (n + 0.5)) from:",
                              "details" : [
                                {
                                  "value" : 28338,
                                  "description" : "n, number of documents containing term",
                                  "details" : [ ]
                                },
                                {
                                  "value" : 97812,
                                  "description" : "N, total number of documents with field",
                                  "details" : [ ]
                                }
                              ]
                            },
                            {
                              "value" : 0.80291224,
                              "description" : "tf, computed as freq / (freq + k1 * (1 - b + b * dl / avgdl)) from:",
                              "details" : [
                                {
                                  "value" : 1.0,
                                  "description" : "freq, occurrences of term within document",
                                  "details" : [ ]
                                },
                                {
                                  "value" : 0.3,
                                  "description" : "k1, term saturation parameter",
                                  "details" : [ ]
                                },
                                {
                                  "value" : 0.75,
                                  "description" : "b, length normalization parameter",
                                  "details" : [ ]
                                },
                                {
                                  "value" : 13.0,
                                  "description" : "dl, length of field",
                                  "details" : [ ]
                                },
                                {
                                  "value" : 17.158834,
                                  "description" : "avgdl, average length of field",
                                  "details" : [ ]
                                }
                              ]
                            }
                          ]
                        }
                      ]
                    }
                  ]
                },
                {
                  "value" : 1.0,
                  "description" : "secondaryWeight",
                  "details" : [ ]
                }
              ]
            }
          ]
        }
      }
```

