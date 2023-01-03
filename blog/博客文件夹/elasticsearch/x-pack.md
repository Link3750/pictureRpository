# x-pack

## 什么是x-pack

x-pack是Elastic Stack的拓展插件，它包括安全、警告、健康报告以及统计功能。由于X-pack组件与Elastic无缝衔接，使用者可以任意启用或禁用x-pack的众多功能。

## 使用x-pack开启用户权限控制

### 在es配置文件中加入以下配置：

``` yml
# 开启 xpack
xpack.security.enabled: true
# 开启传输层 ssl加密，不开启服务无法启动
xpack.security.transport.ssl.enabled: true
```

### 重启elasticsearch

``` 
lsof -i:9200 #查找到es的PID
kill -9 [PID] # 杀掉elastic的进程
cd /[the absolute paths storing the elasticsearch] # 跳转到es的启动目录下
su elas # 由于es不支持使用root用户启动，因此得跳转到普通用户下启动es
./elasticsearch # 启动es
```

### 设置密码（自定义或随机）

``` linux
# 自定义密码
./elasticsearch-setup-passwords interactive
# 随机生成密码
./elasticsearch-setup-passwords auto
```

### 这里选择自定义密码

``` 
Enter password for [elastic]: elastic
Reenter password for [elastic]: elastic

Enter password for [apm_system]: elastic
Reenter password for [apm_system]: elastic

Enter password for [kibana_system]: elastic
Reenter password for [kibana_system]: elastic

Enter password for [logstash_system]: elastic
Reenter password for [logstash_system]: elastic

Enter password for [beats_system]: elastic
Reenter password for [beats_system]: elastic

Enter password for [remote_monitoring_user]: elastic
Reenter password for [remote_monitoring_user]: elastic

Changed password for user [apm_system]
Changed password for user [kibana_system]
Changed password for user [kibana]
Changed password for user [logstash_system]
Changed password for user [beats_system]
Changed password for user [remote_monitoring_user]
Changed password for user [elastic]
```

### 设置完成后

在浏览器输入[es存放主机的IP地址] : [端口号]如： ` 192.168.1.75:9200 `

![image-20220309132456883](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20220309132456883.png)

输入 `elastic`的用户和密码

![image-20220309132557636](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20220309132557636.png)

出现这种格式即为登录成功

### kibana登录设置

在kibana配置文件中新增如下配置：

``` 
xpack.security.enabled: true
elasticsearch.username: [user_name]
elasticsearch.password: [password]
```

### 重启kibana

``` 
lsof -i:5601 #查找到es的PID
kill -9 [PID] # 杀掉kebanac的进程
cd /[the absolute paths storing the kibana] # 跳转到es的启动目录下
su elas
./kebana # 启动es
```

### 打开kibana主页

``` 192.168.1.75:5601```

出现这种界面的话即表示设置成功

![image-20220309134337628](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20220309134337628.png)

在此处输入elasticsearch的账户密码即可登录

*注：输入的是es的账号密码，不是kibana的*

