---
title: 原子操作(Atomic Operations)
tags: [概念原理, 多线程]
---

## 概述

原子操作是多线程操作共享内存的情况下产生的概念，当一个操作是原子操作时，这个操作对其它线程来说是不可分割的

## 使用场景

任何时候，任意两个线程并发操作共享内存，只要其中一个线程包含对内存的写入操作，则这两个线程都必须使用原子操作，否则程序的执行会出现非预期结果

换句话说，假如保证在操作共享内存时，总是单线程在操作，则原子操作不是必须的
{:.info}

## 原子读写(Atomic Loads and Stores)

原子读意味着，对共享内存进行读操作，表现得像是一瞬间读取出完整的值。非原子读可能导致torn reads，例如线程A先读取了一个64位变量的低32位，接着线程B完整地修改了这个变量的值，线程A最后再读取变量的高32位。最终结果是线程A读取到的变量值是修改前的低32位拼合修改后的高32位，一个不应该存在的非预期结果

原子写意味着，对共享内存进行写操作，中间结果对其它线程不可观测。非原子写可能导致torn writes，例如线程A先修改了一个64变量的低32位，接着线程B完整地读取了这个变量的值，线程A最后再修改变量的高32位。最终结果是线程B读取到的变量值是修改后的低32位拼合修改前的高32位，一个不应该存在的非预期结果

Java规范中，对64位的非volatile变量（long或double），读和写的操作不保证是原子的。在32位操作系统的环境下，这样的读和写操作不是原子的，但在64位操作系统的环境下是原子的
{:.info}

Java规范保证对volatile的long或double变量的读和写操作是原子的
{:.info}

## 原子RMW(Atomic Read-Modify-Write)

原子RMW可以看作是原子读写的扩展，不可分割地对共享内存进行读、改、写一系列的操作。原子RMW的通常由CAS(Compare-and Swap)操作来实现

CAS操作过程如下，先读取旧值，与预期值进行对比，若与预期值相等，代表值未被其它线程修改，则将新值写入内存。相反，若与预期值不相等，则不进行任何操作

CAS操作的原子性由CPU对应指令的原子性来保证

## 操作非原子的原因

* 操作由多条CPU指令组成
* 单条CPU指令在执行时转化为多条指令执行
* 一个操作不能保证在所有处理器上执行都是原子的，只能假设这个操作非原子

现代处理器，很多操作都已经保证是原子的
{:.info}

## 参考文档

* [https://preshing.com/20130618/atomic-vs-non-atomic-operations/](https://preshing.com/20130618/atomic-vs-non-atomic-operations/)
* [https://preshing.com/20150402/you-can-do-any-kind-of-atomic-read-modify-write-operation/](https://preshing.com/20150402/you-can-do-any-kind-of-atomic-read-modify-write-operation/)