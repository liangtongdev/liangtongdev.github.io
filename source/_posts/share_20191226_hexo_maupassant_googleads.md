---
layout:     post
title:      Hexo Blog 添加 Google Adsense
date:       2019-12-26 11:40:27
author:     liangtong
catalog: true
categories: 分享
tags: hexo

---

我个人的[Hexo主题博客](http://www.liangtong.site/) 使用的是**maupassant**主题。目前托管在[github仓库](https://github.com/liangtongdev/liangtongdev.github.io)上。3个branch内容安排如下：

+ master 博客仓库
+ hexo 主题博客源码
+ Jekyll 主题博客源码

申请注册Google Adsense账号此处不在说明。直接官网上傻瓜式操作即可。下面仅仅分享一下具体的设置步骤，方便自己以后备查。

**不懂的话，可以将源码下载通过比对文件即可。推荐使用BeyondCompare工具。**

### 主题配置文件

*_config.yml* 文件中，添加以下配置开关

```yml config
show_ad_post: true  ##设置为 true
```

### 页面Head配置

主题目录 *maupassant/layout/_partial/head.pug* 文件中，添加以下配置
```
    if theme.show_ad_post
      script(async='', src='//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js')
      script.
        (adsbygoogle = window.adsbygoogle || []).push({google_ad_client: "ca-pub-xxxxxx",enable_page_level_ads: true});
```
代码不需要自己写，将谷歌给的js代码使用在线转换工具[html2jade](https://www.html2jade.org/)生成即可。 **ca-pub-XXXXX** 替换成自己的账号就行了

### 网站ads.txt文件配置

根据Google Adsense的要求，将ads.txt文件放到博客根目录下即可。

### 经验

多写些原创文章，容易通过审核。如果博客内容不够丰富，可能一次无法审核通过。多写一些再提交申请就好了。
