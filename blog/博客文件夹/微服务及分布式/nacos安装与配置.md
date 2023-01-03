# nacos

## 安装与配置

**测试环境nacos放置在` 192.168.0.82`服务器`/nacos/nacos`目录中**

**生产环境待定**

#### 新建数据库

1. 在mysql中新增一个数据库nacos_config
2. 将/nacos/nacos/conf/nacos-mysql.sql的数据库文件在nacos_config库中运行

#### 修改nacos的配置文件：

1. 打开配置文件` vim /nacos/nacos/conf/application.properties`
2. 修改配置文件

``` properties
db.url.0=jdbc:mysql://127.0.0.1:3306/nacos_config?characterEncoding=utf8&connectTimeout=1000&socketTimeout=3000&autoReconnect=true&useUnicode=true&useSSL=false&serverTimezone=UTC
db.user=nacos
db.password.0=nacos
```

3. 修改nacos启动文件
   1. 打开启动文件 `vim /nacos/nacos/startup.sh`
   2. 将` export MODE="cluster"修改为 export MODE="standalone"

#### 启动nacos

 ` nohup ./startup.sh`

#### 打开nacos主页

`192.168.0.82:8848/nacos`

![image-20220423135938412](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20220423135938412.png)

## nacos使用

![image-20220423152726806](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20220423152726806.png)

我们只使用到nacos的配置中心，因此只讲这块的内容

* 命名空间：命名空间主要用来实现隔离，默认的命名空间是public。我们有三个项目，因此可以设置3个命名空间，来分别管理各个项目的配置。

![image-20220423153908428](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20220423153908428.png)

* DataId：项目用来获取配置文件的标识
* Group：将配置文件分组存放
* 配置样式：不同的配置文件可以对应不同的样式*** 选择不同的样式只是改变配置内容的显示以及提示***
* 配置内容：项目启动所需要的配置文件内容