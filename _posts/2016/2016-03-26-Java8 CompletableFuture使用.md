---
title: Java8 CompletableFuture使用
tags: [Java]
---

## 准备Executor

这里以`ThreadPoolExecutor`为例

```java
ThreadPoolExecutor threadPoolExecutor = new ThreadPoolExecutor(5, 5, 1,
    TimeUnit.SECONDS, new ArrayBlockingQueue<>(10),
    Executors.defaultThreadFactory(),
    new ThreadPoolExecutor.CallerRunsPolicy());
```

## 异步执行任务

执行无返回值任务

```java
CompletableFuture.runAsync(() -> System.out.println("a"), threadPoolExecutor);
```

执行有返回值任务

```java
CompletableFuture<String> bFuture = CompletableFuture.supplyAsync(() -> "b", threadPoolExecutor);  
System.out.println(bFuture.get());
```

## 任务执行完成后处理

`thenApply`方法，对任务执行结果进行函数变换

```java
CompletableFuture<String> bFuture = CompletableFuture.supplyAsync(() -> "b", threadPoolExecutor)
    .thenApply(String::toUpperCase);
```

`handle`方法，对任务执行结果进行函数变换，包括正常结果与异常

```java
CompletableFuture<String> bFuture = CompletableFuture.supplyAsync(() -> "b", threadPoolExecutor)
    .handle((result, exception) -> {  
        if (result != null) {  
            return result;  
        } else {  
            return exception;  
        }  
});
```

`exceptionally`方法，当任务执行抛出异常时，对对任务执行结果进行函数变换

```java
bFuture.exceptionally(exception -> {  
    return exception.getMessage();  
});
```

`thenAccept`方法，消费任务执行结果

```java
CompletableFuture<String> bFuture = CompletableFuture.supplyAsync(() -> "b", threadPoolExecutor)
    .thenAccept(System.out::println);
```

`whenComplete`方法，消费任务执行结果，包括正常结果与异常

```java
CompletableFuture<String> bFuture = CompletableFuture.supplyAsync(() -> "b", threadPoolExecutor)
    .whenComplete((result, exception) -> {  
        System.out.println(result);
        System.out.println(exception.getMessage());
    });
```

`thenRun`方法，任务执行完成后继续执行其它操作

```java
CompletableFuture<String> bFuture = CompletableFuture.supplyAsync(() -> "b", threadPoolExecutor)
    .thenRun(() -> System.out.println("c"));
```

`thenCompose`方法，任务执行完成后继续执行下一个异步任务

```java
CompletableFuture<String> bFuture = CompletableFuture.supplyAsync(() -> "b", threadPoolExecutor)
    .thenCompose(result -> CompletableFuture.supplyAsync(() -> result + "b", threadPoolExecutor));
```

`thenCombine`方法，并发执行两个异步任务，任务全部完成后对执行结果进行函数变换

```java
CompletableFuture<String> bFuture = CompletableFuture.supplyAsync(() -> "b", threadPoolExecutor)
    .thenCombine(CompletableFuture.supplyAsync(() -> "b", threadPoolExecutor),
        (result1, result2) -> result1 + result2);
```

`allOf`方法，等待多个异步任务全部完成

```java
CompletableFuture<String> bFuture = CompletableFuture.supplyAsync(() -> "b", threadPoolExecutor);
CompletableFuture<String> cFuture = CompletableFuture.supplyAsync(() -> "c", threadPoolExecutor);

CompletableFuture<Void> allFuture = CompletableFuture.allOf(bFuture, cFuture);  
allFuture.join();
```

`anyOf`方法，获取多个异步任务第一个完成的结果

```java
CompletableFuture<String> bFuture = CompletableFuture.supplyAsync(() -> "b", threadPoolExecutor);
CompletableFuture<String> cFuture = CompletableFuture.supplyAsync(() -> "c", threadPoolExecutor);

CompletableFuture<Object> anyFuture = CompletableFuture.anyOf(bFuture, cFuture);  
anyFuture.get();
```