---
title: 解决启动OracleOraDb11g home1TNSListener服务启动后又停止的问题
tags: [Oracle, 问题解决]
---

## 解决方法

与系统host文件配置有关，恢复默认配置可能可以解决问题

这可能是由于网络环境的转换造成的，配置在`listener.ora`存在无法访问的host
打开`listener.ora`配置环境（参考目录路径`D:\oracle\product\11.2.0\client_1\network\admin`）
配置`listener`保留计算机名
```
LISTENER =
  (DESCRIPTION_LIST =
    (DESCRIPTION =
      (ADDRESS = (PROTOCOL = TCP)(HOST = ZGC-20140117CHV)(PORT = 1521))
    )
  )
```
