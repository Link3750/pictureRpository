# Nacos Spring Boot快速开始

## 添加依赖

```xml
<dependencyManagement>
    <dependencies>
        <!-- spring cloud alibaba 2.1.0.RELEASE-->
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-alibaba-dependencies</artifactId>
            <!-- 由于jeesite框架的springboot版本过低，且无法升级，只能将springcloud alibaba 的版本给降至2.0.4.RELEASE-->
            <version>2.1.0.RELEASE</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>

</dependencies>
       <!--nacos-config-->
       <dependency>
           <groupId>com.alibaba.cloud</groupId>
           <artifactId>spring-cloud-starter-alibaba-nacos-config</artifactId>
       </dependency>
       <!--nacos-discovery-->
       <dependency>
           <groupId>com.alibaba.cloud</groupId>
           <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
       </dependency>
</dependencies>
```

## 在本地新建bootstrap.yml文件，并增加以下配置

```yaml
server:
  port: 8988

spring:
  application:
    name: fond_app
  cloud:
    nacos:
      discovery:
        server-addr: 192.168.0.82:8848 #nacos服务注册中心地址
      config:
        server-addr: 192.168.0.82:8848 # 配置中心地址
        file-extension: yml # 指定yml格式的配置
        namespace: 78b6c3fb-1010-4892-918e-a2aa77dd8a00 #命名空间
```

## 在启动类中增加注解

```java
@EnableDiscoveryClient
```





