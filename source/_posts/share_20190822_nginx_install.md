---
layout:     post
title:      Mac环境中nginx的安装与配置
date:       2019-08-22 16:50:12
author:     liangtong
catalog: true
categories: share
tags: nginx

---



#### 下载源文件

http://nginx.org/en/download.html

#### 编译安装

```bash
cd nginx-1.17.3 
./configure 
make
sudo make install
```

#### 修改配置

默认编译后路径/usr⁩/local⁩/nginx⁩，配置文件路径： /usr⁩/local⁩/nginx⁩/⁨conf⁩/nginx.config 

```config
    server {
        listen       8090;
        server_name  localhost;

        #charset koi8-r;

        #access_log  logs/host.access.log  main;

        location / {
            root   html;
            index  index.html index.htm;
        }
```

#### 启动

```bash
 sudo /usr/local/nginx/sbin/nginx -c /usr/local/nginx/conf/nginx.conf
```

#### 访问验证

浏览器中访问以下地址进行验证

http://localhost:8090/

http://localhost:8090/index.html

http://localhost:8090/50x.html

![nginx_start.png](/post/share/nginx/20190822/nginx_start.png)

#### 停止

```bash
ps -ef|grep nginx
```

```bash
    0  7886     1   0  4:57下午 ??         0:00.00 nginx: master process /usr/local/nginx/sbin/nginx -c /usr/local/nginx/conf/nginx.conf  
   -2  7887  7886   0  4:57下午 ??         0:00.00 nginx: worker process  
  501  7908  3004   0  5:02下午 ttys000    0:00.00 grep nginx
```

##### 查找 master 对应的进程ID 

比如上图中的7886

##### 杀死进程

+ 从容停止

```bash
 sudo kill -quit 7886
```

+ 快速停止 1

```bash
sudo kill -int 7886
```

+ 快速停止 2

```bash
sudo kill -term 7886
```

+ 强制停止

```bash
sudo pkill -9 nginx
```

#### 重启

##### 验证nginx配置文件是否正确

```bash
cd /usr/local/nginx/sbin/
sudo ./nginx -t
```

结果

```bash
nginx: the configuration file /usr/local/nginx/conf/nginx.conf syntax is ok
nginx: configuration file /usr/local/nginx/conf/nginx.conf test is successful
```

##### 重启nginx服务

```bash
sudo ./nginx -s reload
```

