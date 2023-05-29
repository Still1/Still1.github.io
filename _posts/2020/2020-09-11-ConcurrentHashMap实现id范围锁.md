---
title: ConcurrentHashMap实现id范围锁
tags: [软件开发, Java, 多线程]
---

## 单次获取

```java
public class Test() {
    private final ConcurrentHashMap<Integer, Boolean> concurrentHashMap = new ConcurrentHashMap<>();

    public void doSomethingWithLock() {
        Integer id = 1;
        if (concurrentHashMap.putIfAbsent(id, Boolean.TRUE) == null) {
            // 获取锁成功
            try {
                // 操作与id为1的相关数据
            } finally {
                // 释放锁
                concurrentHashMap.remove(id);
            }
        }
    }
}
```

## 不限次数尝试获取

```java
public class Test() {
    private final ConcurrentHashMap<Integer, Boolean> concurrentHashMap = new ConcurrentHashMap<>();

    public void doSomethingWithLock() {
        Integer id = 1;
        while (true) {
            if (concurrentHashMap.putIfAbsent(id, Boolean.TRUE) == null) {
                // 获取锁成功
                try {
                    // 操作与id为1的相关数据
                } finally {
                    // 释放锁
                    concurrentHashMap.remove(id);
                }
                break;
            }
        }
    }
}
```

## 增加等待锁时间

```java
public class Test() {
    private final ConcurrentHashMap<Integer, Boolean> concurrentHashMap = new ConcurrentHashMap<>();

    @SneakyThrows
    public void doSomethingWithLock() {
        Integer id = 1;
        while (true) {
            if (concurrentHashMap.putIfAbsent(id, Boolean.TRUE) == null) {
                // 获取锁成功
                try {
                    // 操作与id为1的相关数据
                } finally {
                    // 释放锁
                    concurrentHashMap.remove(id);
                }
                break;
            } else {
                // 获取锁失败
                Thread.sleep(1000);
            }
        }
    }
}
```

## 有限次数尝试获取

```java
public class Test() {
    private final ConcurrentHashMap<Integer, Boolean> concurrentHashMap = new ConcurrentHashMap<>();

    @SneakyThrows
    public void doSomethingWithLock() {
        int tryTimes = 10;
        int i = 0;
        Integer id = 1;
        while (true) {
            if (concurrentHashMap.putIfAbsent(id, Boolean.TRUE) == null) {
                // 获取锁成功
                try {
                    // 操作与id为1的相关数据
                } finally {
                    // 释放锁
                    concurrentHashMap.remove(id);
                }
                break;
            } else {
                // 获取锁失败
                if (i == tryTimes - 1) {
                    break;
                }
                i++;
                Thread.sleep(1000);
            }
        }
    }
}
```