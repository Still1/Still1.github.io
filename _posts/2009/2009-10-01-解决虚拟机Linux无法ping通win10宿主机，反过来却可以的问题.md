---
title: 解决虚拟机Linux无法ping通win10宿主机，反过来却可以的问题
tags: [操作系统, 问题解决]
---

## 问题原因

win10防火墙默认禁止

## 解决方法

控制面板 / 系统和安全 / Windows 防火墙
高级设置->入站规则->文件和打印机共享(回显请求 - ICMPv4-In) 专用 公用 启用规则