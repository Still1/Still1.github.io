---
title: 解决SVN无法提交 checksum mismatch occurred的问题
tags: [SVN, 问题解决]
---

## 问题原因

SVN地址迁移后出现的问题

![](https://oliver-blog.oss-cn-shenzhen.aliyuncs.com/20240405064357.png)

## 解决方法

1 备份你提交不上的代码或文件夹
2 删除你原来提交不上的代码或文件夹刷新 提交svn 删除你原来的代码或文件夹
3 原来的代码文件或文件夹放回去  重新提交就可以了。

SVN提交记录将会消失