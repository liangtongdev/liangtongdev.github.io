---
layout:     post
title:      Mac下使用Hexo搭建GitHub博客
date:       2017-08-04 14:25:19
author:     liangtong
categories: share
tags: hexo

---



### 创建GitHub账号   
  注册[Github](https://github.com/)账号，建立个人博客仓库(Repository)，不在赘述。 

### 安装Git工具
  终端直接执行 *git* 命令，或者通过安装GitHub客户端。

### 安装Node和NPM  
  打开终端，通过以下命令安装：    
```Shell
$ brew install node
```

### 安装Hexo  
  可以参照Hexo在[Github](https://github.com/hexojs/hexo)上的官方介绍，在终端依次执行以下命令进行：   
```Shell
$ npm install hexo-cli -g
```    
  安装成功后，可以通过以下命令进行查看   
```Shell
$ hexo
```  

<!-- more -->

### 快速使用博客   

通过在终端输入以下命令执行初始化
```Shell
$ hexo init blog_name
$ cd blog_name
```  
修改Hexo配置文件_config.yml  

```Script
# Site
title: liangtong
subtitle: 
description: Keep hungry keep foolish！
author: liangtong
language: zh-Hans
timezone:
```  

通过在终端输入以下命令新建博客文章
```Shell
$ hexo new "Hello Hexo"
```  
通过在终端输入以下命令生成静态文件
```Shell
$ hexo generate
``` 
通过在终端输入以下命令启动服务
```Shell
$ hexo server
```  

### 更换Hexo主题   
这里使用GitHub上排名最靠前的主题[NexT](https://github.com/iissnan/hexo-theme-next)，在终端输入以下命令进行主题下载：   
```Shell
$ cd blog_name
$ git clone https://github.com/iissnan/hexo-theme-next themes/next
```     
参照[NexT官方指导网站](http://theme-next.iissnan.com/getting-started.html)，对博客进行个性化配置，常用设置包括：   
 + 主题设定   
 + 标签   
 + 分类   
 + 侧边栏   
 + 打赏   
 + 友情链接
 + 动画效果
 + 评论分享   

### 配置Hexo部署  

在Hexo配置文件中完善部署信息，即你的Github个人博客地址信息：   

```Script
# Deployment
## Docs: https://hexo.io/docs/deployment.html
deploy:
type: git
repo: https://github.com/l900416/l900416.github.io.git
branch: master
```    

### 发布Hexo网站到GitHub   
通过终端输入以下命令将Hexo博客发布到Github上：   
```Shell
$ hexo g
```  
此步骤如果出现错误 * Deployer not found: git * ：   
 + 检查Hexo配置的部署文件    
 + 终端输入命令：
    - > npm install hexo-deployer-git --save 

### Hexo在GitHub上的维护    
说下本人的做法，创建新的Branch，命名为hexo，用于存放博客源码，master用于个人博客。这样使用一个仓库来维护。

### Hexo与Jekyll不同之处   
 + Hexo博客文件需要单独维护。对应GitHub仓库中的文件是通过Hexo部署进去的。
 + Jekyll博客文件直接上传到GitHub博客仓库中即可。











