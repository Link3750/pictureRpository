# Linux单节点ES、Kibana升级以及数据迁移

## 升级

*注意：此处升级只是小版本升级，即7. X. X低版本升级到7. X. X高版本，跨大版本升级由于索引不一定通用，因此不在本次文档讨论范围*

升级较为简单，只需下载好升级的版本，并停掉旧版本，开启新版本即可。

### es升级

下载命令：(下载时自行替换成所需的版本)

``` 
ES下载：wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-7.16.2-linux-x86_64.tar.gz
```

解压命令：（解压文件夹自行确定）

```
解压ES：tar -zxvf elasticsearch-7.16.2-linux-x86_64.tar.gz -C /usr/web/elasticsearch
```

关闭原有es,将原es的配置内容转复制到新版本配置文件中，并打开新版本

*注：如果es没安装lsof工具的话先安装该工具：` yum install lsof `*

``` 
lsof -i:<es对应端口号>
```

找到正在运行程序pid并关闭

```
kill -15 <pid>
```

移动到es对应的文件夹内

``` 
cd /usr/web/elasticsearch/elasticsearch-7.16.2-linux-x86_64/bin
```

启动es：

``` 
./elasticsearch
```

启动成功则升级完成

*注：es不允许使用root账号运行程序，测试服务中已经创建好了一个elas用户来运行 elasticSearch。

### Kibana升级

kibana升级方式与es类似，此处仅作简单介绍

``` 
Kibana下载：wget https://artifacts.elastic.co/downloads/kibana/kibana-7.16.2-linux-x86_64.tar.gz
```

``` 
解压kibana：tar -zxvf kibana-7.16.2-linux-x86_64.tar.gz -C /usr/local/kibana
```

关闭老版本kibana，打开新版本的

## 数据迁移

此处采用的数据迁移方式是建快照来迁移

### 配置文件修改

打开旧版本配置文件：在es文件夹内的 config/文件夹下

```
vim /usr/web/elasticsearch/elasticsearch-7.16.2-linux-x86_64/config/elasticsearch.yml
```

进入编辑模式，并增加如下配置

```
path.repo: /usr/web/es_backup/data
```

*注：es_backup/data文件夹为快照存放仓库所在的文件夹，事先自行创建好

### 创建仓库

重启es，并打开kibana，找到开发者工具

![image-20220103142458700](https://raw.githubusercontent.com/Link3750/pictureRpository/main/pic/202301032044966.png)

在开发者工具中创建仓库：

``` json
PUT _snapshot/backup_repository
{
  "type": "fs",
  "settings": {
    "location": "/usr/web/es_backup/data"
  }
}
```

1. backup_repository：创建的仓库名，可以自定义
2. type的类型  fs：共享类型
3. location: 仓库地址，为elasticsearch配置文件中path.repo定义的地址

返回值为：

``` json
{
  "acknowledged" : true
}
```

则表示创建成功

查看某一仓库命令：` GET _snapshot/backup_repository `

查看所有仓库命令：` GET _cat/repositories?v`

### 创建快照

创建快照命令： 

``` json
PUT _snapshot/backup_repository/snapshot_1?wait_for_completion=true
{
  "indices": "fond_publish,fond_order,fond_report,fond_salesmen_performance",
  "ignore_unavailable": true,
  "include_global_state": false
}
```

1. backup_repository为仓库名
2. snapshot_1为快照名
3. wait_for_completion为是否等待创建完成
4. indices为需要创建的索引，多个索引之间用“,”连接，*中间不允许出现空格*
5. include_global_state：防止集群的全局状态被作为快照的一部分存储起来
6. ignore_unavailable：忽略不存在的索引

返回值为

```json
{
  "snapshot" : {
    "snapshot" : "snapshot_1",
    "uuid" : "xClYBOONQaW1LLtJ9vyv5Q",
    "repository" : "backup_repository",
    "version_id" : 7160299,
    "version" : "7.16.2",
    "indices" : [
      "fond_publish",
      "fond_order",
      "fond_salesmen_performance",
      "fond_report"
    ],
    "data_streams" : [ ],
    "include_global_state" : false,
    "state" : "SUCCESS",
    "start_time" : "2022-01-03T06:33:22.337Z",
    "start_time_in_millis" : 1641191602337,
    "end_time" : "2022-01-03T06:33:22.745Z",
    "end_time_in_millis" : 1641191602745,
    "duration_in_millis" : 408,
    "failures" : [ ],
    "shards" : {
      "total" : 4,
      "failed" : 0,
      "successful" : 4
    },
    "feature_states" : [ ]
  }
}

```

表示创建成功

查看快照命令：` GET _snapshot/backup_repository/snapshot_2 `

### 新版本es恢复数据

在新版本配置文件中增加配置：path.repo: /usr/web/es_backup/data

*注：该地址需要与旧版本仓库地址一致*

关闭旧版本es、kibana打开新版本es、kibana

打开kibana的开发者工具

创建仓库：

``` json
PUT _snapshot/backup_repository
{
  "type": "fs",
  "settings": {
    "location": "/usr/web/es_backup/data"
  }
}
```

之后可以恢复数据

``` json
POST _snapshot/backup_repository/snapshot_1/_restore
{
  "indices": "fond_publish,fond_order,fond_report,fond_salesmen_performance",
  "ignore_unavailable": true
}
```

返回值为：

```json
{
  "accepted" : true
}
```

则数据恢复成功

## 操作过程中出现的问题

### es启动过程中报错

1. ``` 
   [2022-01-03T15:14:53,262][ERROR][o.e.b.ElasticsearchUncaughtExceptionHandler] [fond3] uncaught exception in thread [main]
   org.elasticsearch.bootstrap.StartupException: java.lang.RuntimeException: can not run elasticsearch as root
   ```

   es不允许以root用户启动，需要新建用户运行es

2.  ``` 
    2019-03-01 21:52:02,153 main ERROR Unable to invoke factory method in class org.apache.logging.log4j.core.appender.RollingFileAppender for element RollingFile: java.lang.IllegalStateException: No factory method found for class org.apache.logging.log4j.core.appender.RollingFileAppender java.lang.IllegalStateException: No factory method found for class org.apache.logging.log4j.core.appender.RollingFileAppender
    ```

   该用户没有操作log4j日志的权限需要使用root用户将权限给加上操作方式：

   首先切换到root用户

   之后：使用命令`chown elas /usr/web/elasticsearch/elasticsearch-7.16.2-linux-x86_64/logs`

3. 还有类似的没有操作权限的问题，只需要使用类似的方法将相应的目录操作权限添加到elas用户下即可

## kibana启动过程报错

1. Kibana should not be run as root.  Use --allow-root to continue.

   kibana也不允许使用root用户操作，不过kibana有种临时解决方法，即在启动命令` nohup ./kibana`命令后添加上 ` --allow-root`即可在单次启动中使用root用户