---
title: Spring MVC异常统一处理
tags: [软件开发, Java, Spring, Spring MVC]
---

## 代码实例

所有其它的`Controller`抛出的异常，都会先经过`ExceptionHandlingController`，查看是否有对应异常的处理方法。可使用`@ExceptionHandler`注解，或通过方法的参数类型指明处理的异常类型。若有多个处理方法匹配，则由最精确的匹配方法处理

若`ExceptionHandlingController`没有定义对应的异常处理方法，则交给Spring MVC的默认异常处理类`DefaultHandlerExceptionResolver`处理

```java
@RestControllerAdvice  
public class ExceptionHandlingController {  
    @ResponseStatus(HttpStatus.BAD_REQUEST)  
    @ExceptionHandler(HttpMessageNotReadableException.class)  
    public ResponseEntity handleMessageNotReadableException() {  
        return ResponseEntity.wrapMessageResponseEntity(HttpStatus.BAD_REQUEST.getReasonPhrase());  
    }  
  
    @ResponseStatus(HttpStatus.BAD_REQUEST)  
    @ExceptionHandler(MethodArgumentNotValidException.class)  
    public ResponseEntity handleMethodArgumentNotValidException() {  
        return ResponseEntity.wrapMessageResponseEntity(HttpStatus.BAD_REQUEST.getReasonPhrase());  
    }  
  
    @ResponseStatus(HttpStatus.INTERNAL_SERVER_ERROR)  
    @ExceptionHandler(Exception.class)  
    public ResponseEntity handleException() {  
        return ResponseEntity.wrapMessageResponseEntity(HttpStatus.INTERNAL_SERVER_ERROR.getReasonPhrase());  
    }  
}
```