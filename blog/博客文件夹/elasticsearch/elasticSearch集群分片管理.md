# elasticSearch集群分片管理

## 分片定义

### 主分片

1. 因为es是分布式搜索引擎，所以一个索引上的数据通常都会分解成不同的部分，存储于不同的节点中，这些数据就是分片。
2. 主分片能处理索引请求以及查询请求
3. es会自动管理和组织分片，并在必要时对分片数据进行在平衡分配，用户基本不需要担心分片的处理细节

### 副本分片

1. 副本分片是主分片的备份。
2. 副本分片也能处理查询请求，但无法处理索引请求
3. 在主分片丢失或无法使用时，副本分片会升级为主分片，并承担主分片相应的功能

## 设置分片

1. 主分片只能在创建索引时设置，一但索引创建完成便不再允许修改主分片数量。如果需要调整主分片数量，只能新建索引，并使用`reindex`对数据重新索引

2. 副本分片可以在线修改

3. 设置主分片数可以在创建索引的settings时使用` index.number_of_shards: [number] `来设置主分片数量

4. 副本分片数可以在创建索引的settings时使用` index.number_of_replicas: [number]`来设置副本分片数量，也可以使用

   ``` json
   PUT [index]/_settings
   {
     "number_of_replicas": [number]
   }
   ```

   来随时修改副本数量

## 查看分片数量

` GET _cat/shards/[index]?v`用来查看分片数量

### 运行结果

``` 
index              shard prirep state       docs store ip        node
demo_index         0     r      STARTED 32255401 8.6gb 127.0.0.1 node-2
demo_index         0     p      STARTED 32255401 8.7gb 127.0.0.1 node-0

```

index：查询的索引

shard：分片编号；

prirep：分片类型；p——主分片；r——副本分片

state：分片状态；

docs：文档数量

store：文档大小

ip：ip地址

node：节点名称

## es集群中节点下线时，集群如何保证数据不丢失

1. 如果下线的是主节点，那么集群先从候选节点中挑选出新的主节点，若下线的不是主节点，则跳过该步骤
2. 从可用副本中按顺序挑选出一个副本分片升级为主分片代替下线节点上的主分片
3. 如果节点足够的话，重新创建一个副本分片替代升级的原副本分片
4. 在剩余的节点中平衡分片

### 举例

* 3个es实例(node-0, node-1, node-2)组成的集群中，`demo_index` 索引在每个节点中都有1个主分片和2个副本分片

``` 
index        shard prirep state    docs  store ip        node
demo_index   2     r      STARTED 12392 13.4mb 127.0.0.1 node-2
demo_index   2     r      STARTED 12392 13.6mb 127.0.0.1 node-1
demo_index   2     p      STARTED 12392 13.4mb 127.0.0.1 node-0
demo_index   1     r      STARTED 12218 13.2mb 127.0.0.1 node-2
demo_index   1     p      STARTED 12218 13.2mb 127.0.0.1 node-1
demo_index   1     r      STARTED 12218 13.3mb 127.0.0.1 node-0
demo_index   0     p      STARTED 12262 13.4mb 127.0.0.1 node-2
demo_index   0     r      STARTED 12262 13.3mb 127.0.0.1 node-1
demo_index   0     r      STARTED 12262 13.3mb 127.0.0.1 node-0

```

* 现在将`node-2`节点下线，下线后的节点状况如下：

``` 
index        shard prirep state       docs  store ip        node
demo_index   2     r      STARTED    12392 13.6mb 127.0.0.1 node-1
demo_index   2     p      STARTED    12392 13.4mb 127.0.0.1 node-0
demo_index   2     r      UNASSIGNED                        
demo_index   1     p      STARTED    12218 13.2mb 127.0.0.1 node-1
demo_index   1     r      STARTED    12218 13.3mb 127.0.0.1 node-0
demo_index   1     r      UNASSIGNED                        
demo_index   0     p      STARTED    12262 13.3mb 127.0.0.1 node-1
demo_index   0     r      STARTED    12262 13.3mb 127.0.0.1 node-0
demo_index   0     r      UNASSIGNED                        

```

​	此时可以看到`node-2`节点已经下线，原本在` node-2 `节点中的主分片`shard-p-0`由位于`node-1` 节点中的`shard-r-0`代替（升级为新的`shard-p-0`），而位于`node-0`节点中的`shard-r-1`则变为了`shard-r-0`。由于此时主节点还是3个，但是副本节点变成了1个，此时索引状态变为了`yellow`，即可正常访问，但缺失副本。

* 重新上线`node-2`节点，状况如下：

``` 
index        shard prirep state    docs  store ip        node
demo_index   2     r      STARTED 12392 13.5mb 127.0.0.1 node-2
demo_index   2     r      STARTED 12392 13.6mb 127.0.0.1 node-1
demo_index   2     p      STARTED 12392 13.5mb 127.0.0.1 node-0
demo_index   1     r      STARTED 12218 13.2mb 127.0.0.1 node-2
demo_index   1     p      STARTED 12218 13.2mb 127.0.0.1 node-1
demo_index   1     r      STARTED 12218 13.4mb 127.0.0.1 node-0
demo_index   0     r      STARTED 12262 13.3mb 127.0.0.1 node-2
demo_index   0     p      STARTED 12262 13.3mb 127.0.0.1 node-1
demo_index   0     r      STARTED 12262 13.3mb 127.0.0.1 node-0

```

主节点仍然分配在`node-0`、`node-1`两个节点中，`node-2`节点中的分片全部变为了副本节点。即下线后重新上线的节点并不会复原其本来的主节点，而是在原有的副本分片中同步新数据。此时索引的主节点、副本均已恢复，因此索引状态由`yellow`变回`green`

## 手动修改索引分片

es默认会自动修改索引的分片，一般情况下不需要手动调整

如需手动修改，必须先将自动路由分片关闭：

``` json
PUT _cluster/settings 
{ 
  "persistent": { 
    "cluster.routing.allocation.enable": "none"
  }
}
```

成功后会返回

```json
{
  "acknowledged" : true,
  "persistent" : {
    "cluster" : {
      "routing" : {
        "allocation" : {
          "enable" : "none"
        }
      }
    }
  },
  "transient" : { }
}
```

**注意：主分片与副本分片不能同时处于一个节点中。**

删除副本分片命令：

```json
POST _cluster/reroute
{
  "commands": [
    {
      "cancel": {
        "index": "fond_publish",
        "shard": 2,
        "node": "node-2"
      }
    }
  ]
}
```

移动分片命令：

```json
POST /_cluster/reroute
{
  "commands":[
		{
			"move": {
			"index": "fond_publish",
			"shard": 2,
			"from_node": "node-0",
			"to_node": "node-2"
			}
		}
	]
}
```

移动完成后将自动路由分片开启：

```json
PUT _cluster/settings 
{ 
  "persistent": { 
    "cluster.routing.allocation.enable": "all"
  }
}
```

