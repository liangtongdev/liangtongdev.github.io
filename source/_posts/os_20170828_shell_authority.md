---
layout:     post
title:      修改文件所有者
date:       2017-08-28 11:51:43
author:     liangtong
catalog: true
categories: 操作系统
tags: shell

---

#### chown

  使用超级管理员用户权限时，可以更改文件的所有者和用户组，命令如下：   

 > chown [owner][:[group]] file...

命令将文件更改为第一个参数所表示的所有者 :所在分组。 例如：

```shell
liangtongdeiMac:test_authority liangtong$ sudo chown guest authority_test.txt 
Password:
liangtongdeiMac:test_authority liangtong$ ls -l
total 0
-rw-r--r--  1 Guest  staff  0  8 16 18:43 authority_test.txt
liangtongdeiMac:test_authority liangtong$ 

```

 + 若命令中只包含 *用户名* ，则是把所有者更改为用户。
 + 若命令中包含 *:用户组* ，则将文件的用户组修改。
 + 特别的，若命令中参数格式是 *用户:* ， 即未指明用户组，那么用户组指的是用户所在的分组。

#### chgrp
  使用超级管理员用户权限时，可以使用 *chgrp* 来修改用户组，命令与chown相似：

 > chgrp group file




