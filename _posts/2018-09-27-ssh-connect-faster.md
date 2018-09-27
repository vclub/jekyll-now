---
layout: post
title: ssh连接缓慢优化
tags: [Jenkins]
---

# ssh连接缓慢优化

```vi /etc/ssh/sshd_config ```

关闭 SSH的DNS反解析, 添加下面一行

```UseDNS no```

/etc/init.d/sshd restart 重启sshd进程使配置生效