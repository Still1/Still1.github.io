---
title: 解决SSH连接Socket error Event 32 Error 10053错误的问题
tags: [操作系统, 问题解决]
---

## 解决方法

可能需要
```shell
chmod 644 sshd_config
```

修改文件
`/etc/ssh/sshd_config`

```
UseDNS no
ClientAliveInterval 60
```

重启sshd
```shell
systemctl restart sshd
```

也可能是本地网络问题导致的