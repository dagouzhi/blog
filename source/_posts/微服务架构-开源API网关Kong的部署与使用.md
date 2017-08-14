---
title: 微服务架构-开源API网关Kong的部署与使用
date: 2016-12-17 00:47:09
tags: kong,api
---
[原文 酱酱酱子](http://heqin.blog.51cto.com/8931355/1865665)

### 前言：
   微服务架构是现在很火热的技术话题，其本质是将庞大而复杂的系统，拆分成微小的服务模块，使服务之间实现解耦，能够实现各模块的独立开发、升级、部署。大大降低了系统的运维及开发难度。

 然而由于服务的拆分，客户端可能同时需要和多个服务进行交互，随着微服务规模的增大，这样的交互模式在性能和管理上都有很大的不便。那么基于微服务的客户端如何能够更好的去访问这些独立的服务呢，这时我们就需要一个统一的入口来向外提供服务。这就是我们所说的API网关，

  API Gateway是一个在客户端和服务之间的中间人，客户端不用直接访问服务器，而是通过API Gateway来传递中间消息。API Gateway能够实现负载均衡、缓存、访问控制、API 计费监控等等功能。下面为网上关于API Gateway的图。

  Kong 是由Mashape公司开发的一款API Gateway的软件，Kong是基于nginx开发，用来接收客户端的API请求，同时还需要一个数据库来存储操作数据。写这篇文章时Kong的最新版是0.9.3，其支持数据库为PostgreSQL 9.4+ 和Cassandra 2.2.x .

  一：安装
centos
（1）：安装kong
```
$ sudo yum install epel-release
$ sudo yum install kong-0.9.3.*.noarch.rpm --nogpgcheck
```

or
Download kong-0.9.3.el7.noarch.rpm
```
$ wget kong-0.9.3.el7.noarch.rpm
```
（2）：配置数据库
kong支持 PostgreSQL 9.4+ 和Cassandra 2.2.x .
如果使用PostgreSQL数据库，请创建用户和对应的数据库
```
$ CREATE USER kong; CREATE DATABASE kong OWNER kong;
```
（3）：启动

```
$ kong start
# Kong is running
$ curl 127.0.0.1:8001
```
Kong启动后，会分别监听8000端口和8001端口。8000端口是用来提供服务，8001是用来对API进行管理。


docker
（1）:启动数据库
cassandra
```
$ docker run -d --name kong-database \
              -p 9042:9042 \
              cassandra:2.2
```

OR PostgreSQL

```
$ docker run -d --name kong-database \
              -p 5432:5432 \
              -e "POSTGRES_USER=kong" \
              -e "POSTGRES_DB=kong" \
              postgres:9.4
```

（2）：启动kong

```
$ docker run -d --name kong \
              --link kong-database:kong-database \
              -e "KONG_DATABASE=cassandra" \
              -e "KONG_CASSANDRA_CONTACT_POINTS=kong-database" \
              -e "KONG_PG_HOST=kong-database" \
              -p 8000:8000 \
              -p 8443:8443 \
              -p 8001:8001 \
              -p 7946:7946 \
              -p 7946:7946/udp \
              kong
```

附上docker-compose.yml

```
kong-database:
  image: postgres:9.4
  ports:
  - 5432:5432/tcp
  environment:
  - POSTGRES_USER=kong
  - POSTGRES_DB=kong
kong:
  image: kong
  links:
  -  kong-database:kong-database
  environment:
  - KONG_DATABASE=postgres
  - KONG_CASSANDRA_CONTACT_POINTS=kong-database
  - KONG_PG_HOST=kong-database
  ports:
  - 8000:8000/tcp
  - 8443:8443/tcp
  - 8001:8001/tcp
  - 7946:7946/tcp
  - 7946:7946/udp
```

二：使用kong
（1）：添加api到kong

```
$ curl -i -X POST \
--url http://localhost:8001/apis/ \
--data 'name=baidu' \
--data 'upstream_url=http://baidu.com/' \
--data 'request_host=baidu.com'
--url：8001端口是Kong的管理端口。
```
upstream_url：提供服务的后端url。

request_path：使用path参数时，加入该路径的服务接口。

request_host 使用这个参数时，将加入该host的所有服务接口：


（2）:查询已经添加的API

```
$curl localhost:8001/apis/
```

（3）:访问API

```
$curl -i -X POST --url http://localhost:8000/ --header 10.100.55.1
```

（4）:删除API
根据API_NAME or API_ID来删除指定api
```
$curl -i -X DELETE localhost:8001/apis/00f90ca9-cf2d-4830-c842-3b90f6cd08af
$curl -i -X DELETE localhost:8001/apis/test1
```
（5）:添加实例
实例1  (request_path限制path内的接口)
URL:
http://10.100.55.1/hello1/index.html
添加:

```
$curl -i -X POST --url http://localhost:8001/apis/ \
--data name=test1 \
--data 'upstream_url=http://10.100.55.1' \
--data 'request_path=/hello1/'
```

访问接口:
```
$curl -i -X GET --url http://localhost:8000/hello1/
```
所谓的request_path，就是固定只能访问hello1下的内容，如果该host的www目录下还有hello2、hello3、那么是不能通过网关去访问这个目录下的内容，例如下面是访问不到的，因为没有加入到Kong中。
```
$curl -i -X GET --url http://localhost:8000/hello2/
```

实例2  (request_host可以访问host所有接口)
URL:
http://10.100.55.1
添加:
```
$ curl -i -X POST --url http://localhost:8001/apis/ \
--data name=test2 \
--data 'upstream_url=http://10.100.55.1' \
--data 'request_host=10.100.55.1'
```
访问接口:
使用request_host后，该host所有api都加入到Kong中，下面的都能够通过网关访问。
```
$curl -i -X GET --url http://localhost:8000/hello1  --header host:10.100.55.1
$curl -i -X GET --url http://localhost:8000/hello2 --header host:10.100.55.1
```
实例3 (request_host  port:8080)
URL:
http://10.100.55.2:8080
添加:
```
$curl -i -X POST --url http://localhost:8001/apis/ \
--data  name=test3  \
--data 'upstream_url=http://10.100.55.2:8080' \
--data 'request_host=10.100.55.2'
```
访问接口:
```
$curl -i -X GET --url http://localhost:8000/ --header host:10.100.55.2
```
实例4 （复杂url的添加和访问）
URL:
http://10.100.55.3:8000/opj/list?serviceId=box&c=nanjing
添加:
1
$ curl -i -X POST --url http://localhost:8001/apis/ --data 'name=test4' --data 'upstream_url=http://10.100.55.3:8000/' --data 'request_path=/opj/list'
访问接口:
```
$ curl -i -X GET --url http://localhost:8000/opj/list?serviceId=box&c=nanjing
```


三：创建认证
（1）给API配置pulgin认证
1：添加api
```
$curl -i -X POST --url http://localhost:8001/apis/ --data 'name=test5' --data 'upstream_url=http://10.100.55.1/' --data 'request_host=10.100.55.1'
$curl -i -X GET --url http://localhost:8000/ --header host:10.100.55.1
```
访问正常
```
Connect  Success...
```
2：添加plugin认证
```
$curl -i -X POST --url http://localhost:8001/apis/test5/plugins/ --data 'name=key-auth'
$curl -i -X GET --url http://localhost:8000/ --header host:10.100.55.1
```
访问失败
```
HTTP/1.1 401 Unauthorized
WWW-Authenticate: Key realm="kong"
Server: kong/0.9.3
{"message":"No API key found in headers or querystring"}
```
（2）添加用户
1：创建用户
```
$curl -i -X POST --url http://localhost:8001/consumers/ --data "username=heqin"
{"username":"heqin","created_at":1477382339000,"id":"8e6273c9-f332-4d68-b74c-73ae9f82f150"}
```
2：给用户创建key
```
$curl -i -X POST --url http://localhost:8001/consumers/heqin/key-auth/ --data 'key=helloworld'
{"created_at":1477382483000,"consumer_id":"8e6273c9-f332-4d68-b74c-73ae9f82f150","key":"helloworld","id":"62c0d640-b1bd-4f3b-aa6e-ba3adaf8ec38"}
```
3：带着key去访问
```
$curl -i -X GET --url http://localhost:8000/  --header host:10.100.55.1 --header apikey:helloworld
```
访问成功
```
Connect  Success...
```
通过上面的两步，就可以实现对API接口的权限控制。

未完待续。。。。。。。
