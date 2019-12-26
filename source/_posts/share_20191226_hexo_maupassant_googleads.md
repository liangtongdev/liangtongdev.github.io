---
layout:     post
title:      Hexo Blog 添加 Google Adsense
date:       2019-12-26 11:40:27
author:     liangtong
catalog: true
categories: share
tags: hexo

---

我个人的[Hexo主题博客](www.liangtong.site)使用的是**maupassant**主题，目前放在github上。

申请注册Google Adsense账号此处不在说明。直接官网上傻瓜式操作即可。下面仅仅分享一下具体的设置步骤，方便自己以后备查。

#### 主题配置文件

*_config.yml* 文件中，添加以下配置开关

```yml config
show_ad_post: true  ##设置为 true
```

#### Head配置文件
主题目录 *maupassant/layout/_partial/head.pug* 文件中，添加以下配置
```
    if theme.show_ad_post
      script(async='', src='//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js')
      script.
        (adsbygoogle = window.adsbygoogle || []).push({google_ad_client: "ca-pub-xxxxxx",enable_page_level_ads: true});
```
代码不需要自己写，将谷歌给的js代码使用在线转换工具[html2jade](https://www.html2jade.org/)生成即可。 **ca-pub-XXXXX** 替换成自己的账号就行了

#### ads.txt配置文件
根据Google Adsense的要求，将ads.txt文件放到博客根目录下即可。

#### 等待审核

多写些原创文章，容易通过审核。如果博客内容不够丰富，可能一次无法审核通过。多写一些再提交申请就好了。

#### 写在最后

**万水千山总是情，点下广告行不行！**