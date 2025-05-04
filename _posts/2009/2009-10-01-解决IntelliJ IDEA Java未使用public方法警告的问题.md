---
title: 解决IntelliJ IDEA Java未使用public方法警告的问题
tags: [IntelliJ IDEA, 问题解决]
---

## 问题原因

默认情况下，public没使用过的方法会有警告

## 解决方法

解决设置如下：
> Of course you can turn it off.Settings | Inspections | Declaration redundancy | Unused declararation. 
> Turning off this specific inspection will turn off editor highlighting forunused public members. 
> Unused private members will still be highlighted.

关闭后依然会检查private方法
可以通过scope来控制哪部分代码该检查