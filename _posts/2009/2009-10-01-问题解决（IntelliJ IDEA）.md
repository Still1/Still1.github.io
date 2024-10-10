---
title: 问题解决（IntelliJ IDEA）
tags: [IntelliJ IDEA, 问题解决]
---

## IntelliJ IDEA 解决public方法检查是否使用过的问题

默认情况下，public没使用过的方法会有警告

解决设置如下：
> Of course you can turn it off.Settings | Inspections | Declaration redundancy | Unused declararation. 
> Turning off this specific inspection will turn off editor highlighting forunused public members. 
> Unused private members will still be highlighted.

关闭后依然会检查private
可以通过scope来控制哪部分代码该检查

## 解决IDEA jQuery代码unresloved function or method $() 

解决方法：
IDEA里安装jQuery库并引入
![](https://oliver-blog.oss-cn-shenzhen.aliyuncs.com/20240405125726.png)

## IDEA Maven3.6.3无法下载依赖包

问题原因：有可能是IDEA版本 JDK版本 Maven版本 之间的关系不兼容
问题解决：使用Maven3.5.3版本

## IDEA 警告 Not annotated parameter overrides @NonNullApi parameter

解决方法：对应参数加上注解`@SuppressWarnings("NullableProblems")`压制警告

## Maven pom.xml IDEA警告 provides transitive vulnerable dependency

解决方法：加上以下注解

```xml
<!--suppress VulnerableLibrariesLocal, MavenPackageUpdate -->
```

## Spring Initializr新建模块没有Java 8

解决方法：Server URL 修改为https://start.aliyun.com

![](https://oliver-blog.oss-cn-shenzhen.aliyuncs.com/20240519165308.png)