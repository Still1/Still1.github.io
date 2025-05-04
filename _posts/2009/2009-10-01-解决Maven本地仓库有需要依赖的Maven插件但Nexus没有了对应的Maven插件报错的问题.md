---
title: 解决Maven本地仓库有需要依赖的Maven插件但Nexus没有了对应的Maven插件报错的问题
tags: [Maven, 问题解决]
---

## 问题描述

```
Failure to find org.jfrog.maven.annomojo:maven-plugin-anno:jar:1.4.0 in http://myrepo:80/artifactory/repo was cached in the local repository, resolution will not be reattempted until the update interval of MyRepo has elapsed or updates are forced
```

## 解决方法

解决方法：
![](https://oliver-blog.oss-cn-shenzhen.aliyuncs.com/20240405073913.png)

删掉对应插件的此文件，问题解决

http://stackoverflow.com/questions/4856307/when-maven-says-resolution-will-not-be-reattempted-until-the-update-interval-of