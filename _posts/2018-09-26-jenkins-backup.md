---
layout: post
title: Jenkins 简单备份迁移
tags: [Jenkins]
---

Jenkins简单备份迁移

Jenkins主目录下的config.xml以及jobs, tools, users, plugins

jobs 是任务的信息

tools 是工具文件夹

users 是登录用户文件夹

plugins 是jenkins的插件

jobs文件夹下有之前编译的旧的信息,
可以通过如下命令简单的只保留配置,

```zip -r jobs_bk.zip */config.xml```

迁移后,重新加载一下配置就可以了,注意要把目录权限改为jenkins用户

