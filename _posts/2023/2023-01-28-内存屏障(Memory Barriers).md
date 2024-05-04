---
title: 内存屏障(Memory Barriers)
tags: [概念原理, 计算机组成原理]
---

## 概述

内存屏障用于解决运行时产生的内存重排序（又称处理器重排序），通常通过加入CPU屏障指令(fence instructions)来实现加入内存屏障

## 分类

为了便于理解，此处我们将高速缓存看作是内存的延申，先把高速缓存与内存看成是一个整体，并以内存一词并称
{:.warning}

* LoadLoad屏障，重点在于屏障后的内存读取操作，位于屏障后的任意内存读取操作必须晚于屏障前的任意内存读取操作，不保证屏障后的内存读取操作读取到当前内存的最新值，但保证与屏障前的任意内存读取操作相比，能读取到同等新或更新的数据
* StoreStore屏障，重点在于屏障前的内存写入操作，位于屏障前的任意内存写入操作必须早于屏障后的任意内存写入操作，不保证屏障前的内存写入操作什么时候完成，但保证比屏障后的任意内存写入操作先完成
* LoadStore屏障，单纯限制内存操作顺序，位于屏障前的任意内存读取操作必须早于屏障后的任意内存写入操作
* StoreLoad屏障，位于屏障前的任意内存写入操作必须早于屏障后的任意内存读取操作，通过屏障时，保证屏障前的所有内存写入操作已完成，保证屏障后的所有内存读取操作能读取到与通过屏障时同等新或更新的数据

## 实现

前文已提过，内存屏障通常通过加入CPU屏障指令来实现，一个CPU屏障指令一般会对应一个内存屏障组合，再外加一些别的作用。不同架构的CPU会有不同的CPU屏障指令，具体实现也各不相同，但普遍有以下的共同点

* LoadStore屏障不会被单独实现，会与LoadLoad屏障或StoreStore屏障组合在一起
* StoreLoad屏障不会被单独实现，而是实现为全能屏障(full memory fence)，同时兼具LoadLoad、StoreStore、LoadStore、StoreLoad屏障的作用

## 参考文档

* [https://preshing.com/20120710/memory-barriers-are-like-source-control-operations/](https://preshing.com/20120710/memory-barriers-are-like-source-control-operations/)