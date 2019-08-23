---
layout:     post
title:      nginx的主要功能及用法
date:       2019-08-23 11:40:27
author:     liangtong
catalog: true
categories: share
tags: nginx

---

Nginx主要功能：1、反向代理 2、负载均衡 3、HTTP服务器（包含动静分离） 4、正向代理 。

### 反向代理

例如以下配置，本地监听9090端口，接收到请求http://localhost:9090/kanban时，转向localhost:8080服务，一个服务器不同端口代理。

```nginx config
server {
        listen       9090;
        server_name  localhost;
        autoindex on;
        autoindex_exact_size  on;
        autoindex_localtime on;
        #charset utf-8;
        #access_log  logs/host.access.log  main;
        location /kanban {
            proxy_pass http://localhost:8080;
            proxy_set_header Host $host:$server_port;
        }
    }
```

反向代理应该是 Nginx 做的最多的一件事了，什么是反向代理呢，以下是百度百科的说法：反向代理（Reverse Proxy）方式是指以代理服务器来接受internet上的连接请求，然后将请求转发给内部网络上的服务器，并将从服务器上得到的结果返回给 internet 上请求连接的客户端，此时代理服务器对外就表现为一个反向代理服务器。简单来说就是真实的服务器不能直接被外部网络访问，所以需要一台代理服务器，而代理服务器能被外部网络访问的同时又跟真实服务器在同一个网络环境，当然也可能是同一台服务器，端口不同而已。 


### 负载均衡

负载均衡也是 Nginx 常用的一个功能，负载均衡其意思就是分摊到多个操作单元上进行执行，例如：Web服务器、FTP服务器、企业关键应用服务器和其它关键任务服务器等，从而共同完成工作任务。简单而言就是当有2台或以上服务器时，根据规则随机的将请求分发到指定的服务器上处理，负载均衡配置一般都需要同时配置反向代理，通过反向代理跳转到负载均衡。而Nginx目前支持自带3种负载均衡策略。 
负载均衡配置关键字**upstream**

```nginx config
upstream customserver  {
  server 127.0.0.1:8080;
  server 127.0.0.1:8090;
}
```

upstream还可以为每个设备设置状态值

+ down 表示单前的server暂时不参与负载
+ weight 默认为1.weight越大，负载的权重就越大
+ max_fails 允许请求失败的次数默认为1.当超过最大次数时，返回proxy_next_upstream 模块定义的错误
+ fail_timeout max_fails次失败后，暂停的时间
+ backup 其它所有的非backup机器down或者忙的时候，请求backup机器。所以这台机器压力会最轻

#### 轮询（默认）

upstream kanbanser  {
  server 127.0.0.1:8080;
  server 127.0.0.1:8090;
}

server {
        listen       9090;
        server_name  localhost;
        autoindex	on;
        autoindex_exact_size	on;
        autoindex_localtime	on;

        location /kanban {
            proxy_pass http://kanbanser;
            proxy_set_header Host $host:$server_port;
        }
    }

#### 权重

upstream kanbanser  {
  server 127.0.0.1:8080 weight=2;
  server 127.0.0.1:8090 weight=1;
}

#### ip_hash

upstream kanbanser  {
  ip_hash; 
  server 127.0.0.1:8080;
  server 127.0.0.1:8090;
}


### HTTP服务器（包含动静分离）

Nginx本身也是一个静态资源的服务器，当只有静态资源的时候，就可以使用Nginx来做服务器，同时现在也很流行动静分离，就可以通过Nginx来实现，首先看看Nginx做静态资源服务器。

比如以下配置，将http://localhost:9090的以「htm|html」结尾的请求静态资源，从html文件夹下查找。


```nginx config
upstream kanbanser  {
  ip_hash; 
  server 127.0.0.1:8080;
  server 127.0.0.1:8090;
}

server {
        listen       9090;
        server_name  localhost;
        autoindex	on;
        autoindex_exact_size	on;
        autoindex_localtime	on;
        #charset utf-8;
        #access_log  logs/host.access.log  main;
        location /kanban {
            proxy_pass http://kanbanser;
            proxy_set_header Host $host:$server_port;
        }
        location ~ .*\.(htm|html)$ {
            root   html;
        }
    }
```

### 正向代理

所谓正向代理就是内网服务器主动要去请求外网的地址或服务，所进行的一种行为。内网服务---访问--->外网



### 其他

nginx配置文件包含，使用**include** 关键字，例如

```nginx config
    include nginx_test.conf;
```

