---
title: Spring MVC RESTful全局异常处理
tags: [Spring MVC]
---

## 代码实例

```java
package com.oliverclio.languagetrainer.wrap;  
  
import lombok.Data;  
  
@Data  
public class ResponseEntity {  
    public static final String SUCCESS_MESSAGE = "OK";  
  
    private String message;  
    private Object data;  
  
    private ResponseEntity() {  
    }  
  
    public static ResponseEntity wrapDataResponseEntity(Object data) {  
        ResponseEntity responseEntity = new ResponseEntity();  
        responseEntity.setData(data);  
        responseEntity.setMessage(SUCCESS_MESSAGE);  
        return responseEntity;  
    }  
  
    public static ResponseEntity wrapMessageResponseEntity(String message) {  
        ResponseEntity responseEntity = new ResponseEntity();  
        responseEntity.setMessage(message);  
        return responseEntity;  
    }  
  
    public static ResponseEntity wrapSuccessMessageResponseEntity() {  
        ResponseEntity responseEntity = new ResponseEntity();  
        responseEntity.setMessage(SUCCESS_MESSAGE);  
        return responseEntity;  
    }  
}
```

* `@RestControllerAdvice`指定全局异常处理类
* `@ResponseStatus`指定响应状态码
* `@ExceptionHandler`指定匹配的异常类型

```java
package com.oliverclio.languagetrainer.config.exception;  
  
import com.oliverclio.languagetrainer.wrap.ResponseEntity;  
import lombok.extern.slf4j.Slf4j;  
import org.springframework.beans.ConversionNotSupportedException;  
import org.springframework.beans.TypeMismatchException;  
import org.springframework.http.HttpStatus;  
import org.springframework.http.converter.HttpMessageNotReadableException;  
import org.springframework.http.converter.HttpMessageNotWritableException;  
import org.springframework.validation.BindException;  
import org.springframework.web.HttpMediaTypeNotAcceptableException;  
import org.springframework.web.HttpMediaTypeNotSupportedException;  
import org.springframework.web.HttpRequestMethodNotSupportedException;  
import org.springframework.web.bind.MethodArgumentNotValidException;  
import org.springframework.web.bind.MissingPathVariableException;  
import org.springframework.web.bind.MissingServletRequestParameterException;  
import org.springframework.web.bind.ServletRequestBindingException;  
import org.springframework.web.bind.annotation.ExceptionHandler;  
import org.springframework.web.bind.annotation.ResponseStatus;  
import org.springframework.web.bind.annotation.RestControllerAdvice;  
import org.springframework.web.context.request.async.AsyncRequestTimeoutException;  
import org.springframework.web.multipart.support.MissingServletRequestPartException;  
import org.springframework.web.servlet.NoHandlerFoundException;  
  
@RestControllerAdvice  
@Slf4j  
public class ExceptionHandlingController {  
    @ResponseStatus(HttpStatus.BAD_REQUEST)  
    @ExceptionHandler(HttpMessageNotReadableException.class)  
    public ResponseEntity handleMessageNotReadableException(Exception e) {  
        logException(e);  
        return ResponseEntity.wrapMessageResponseEntity(HttpStatus.BAD_REQUEST.getReasonPhrase());  
    }  
  
    @ResponseStatus(HttpStatus.BAD_REQUEST)  
    @ExceptionHandler(MethodArgumentNotValidException.class)  
    public ResponseEntity handleMethodArgumentNotValidException(Exception e) {  
        logException(e);  
        return ResponseEntity.wrapMessageResponseEntity(HttpStatus.BAD_REQUEST.getReasonPhrase());  
    }  
  
    @ResponseStatus(HttpStatus.METHOD_NOT_ALLOWED)  
    @ExceptionHandler(HttpRequestMethodNotSupportedException.class)  
    public ResponseEntity handleHttpRequestMethodNotSupportedException(Exception e) {  
        logException(e);  
        return ResponseEntity.wrapMessageResponseEntity(HttpStatus.METHOD_NOT_ALLOWED.getReasonPhrase());  
    }  
  
    @ResponseStatus(HttpStatus.UNSUPPORTED_MEDIA_TYPE)  
    @ExceptionHandler(HttpMediaTypeNotSupportedException.class)  
    public ResponseEntity handleHttpMediaTypeNotSupportedException(Exception e) {  
        logException(e);  
        return ResponseEntity.wrapMessageResponseEntity(HttpStatus.UNSUPPORTED_MEDIA_TYPE.getReasonPhrase());  
    }  
  
    @ResponseStatus(HttpStatus.NOT_ACCEPTABLE)  
    @ExceptionHandler(HttpMediaTypeNotAcceptableException.class)  
    public ResponseEntity handleHttpMediaTypeNotAcceptableException(Exception e) {  
        logException(e);  
        return ResponseEntity.wrapMessageResponseEntity(HttpStatus.NOT_ACCEPTABLE.getReasonPhrase());  
    }  
  
    @ResponseStatus(HttpStatus.INTERNAL_SERVER_ERROR)  
    @ExceptionHandler(MissingPathVariableException.class)  
    public ResponseEntity handleMissingPathVariableException(Exception e) {  
        logException(e);  
        return ResponseEntity.wrapMessageResponseEntity(HttpStatus.INTERNAL_SERVER_ERROR.getReasonPhrase());  
    }  
  
    @ResponseStatus(HttpStatus.BAD_REQUEST)  
    @ExceptionHandler(MissingServletRequestParameterException.class)  
    public ResponseEntity handleMissingServletRequestParameterException(Exception e) {  
        logException(e);  
        return ResponseEntity.wrapMessageResponseEntity(HttpStatus.BAD_REQUEST.getReasonPhrase());  
    }  
  
    @ResponseStatus(HttpStatus.BAD_REQUEST)  
    @ExceptionHandler(ServletRequestBindingException.class)  
    public ResponseEntity handleServletRequestBindingException(Exception e) {  
        logException(e);  
        return ResponseEntity.wrapMessageResponseEntity(HttpStatus.BAD_REQUEST.getReasonPhrase());  
    }  
  
    @ResponseStatus(HttpStatus.INTERNAL_SERVER_ERROR)  
    @ExceptionHandler(ConversionNotSupportedException.class)  
    public ResponseEntity handleConversionNotSupportedException(Exception e) {  
        logException(e);  
        return ResponseEntity.wrapMessageResponseEntity(HttpStatus.INTERNAL_SERVER_ERROR.getReasonPhrase());  
    }  
  
    @ResponseStatus(HttpStatus.BAD_REQUEST)  
    @ExceptionHandler(TypeMismatchException.class)  
    public ResponseEntity handleTypeMismatchException(Exception e) {  
        logException(e);  
        return ResponseEntity.wrapMessageResponseEntity(HttpStatus.BAD_REQUEST.getReasonPhrase());  
    }  
  
    @ResponseStatus(HttpStatus.INTERNAL_SERVER_ERROR)  
    @ExceptionHandler(HttpMessageNotWritableException.class)  
    public ResponseEntity handleHttpMessageNotWritableException(Exception e) {  
        logException(e);  
        return ResponseEntity.wrapMessageResponseEntity(HttpStatus.INTERNAL_SERVER_ERROR.getReasonPhrase());  
    }  
  
    @ResponseStatus(HttpStatus.BAD_REQUEST)  
    @ExceptionHandler(MissingServletRequestPartException.class)  
    public ResponseEntity handleMissingServletRequestPartException(Exception e) {  
        logException(e);  
        return ResponseEntity.wrapMessageResponseEntity(HttpStatus.BAD_REQUEST.getReasonPhrase());  
    }  
  
    @ResponseStatus(HttpStatus.BAD_REQUEST)  
    @ExceptionHandler(BindException.class)  
    public ResponseEntity handleBindException(Exception e) {  
        logException(e);  
        return ResponseEntity.wrapMessageResponseEntity(HttpStatus.BAD_REQUEST.getReasonPhrase());  
    }  
  
    @ResponseStatus(HttpStatus.NOT_FOUND)  
    @ExceptionHandler(NoHandlerFoundException.class)  
    public ResponseEntity handleNoHandlerFoundException(Exception e) {  
        logException(e);  
        return ResponseEntity.wrapMessageResponseEntity(HttpStatus.NOT_FOUND.getReasonPhrase());  
    }  
  
    @ResponseStatus(HttpStatus.SERVICE_UNAVAILABLE)  
    @ExceptionHandler(AsyncRequestTimeoutException.class)  
    public ResponseEntity handleAsyncRequestTimeoutException(Exception e) {  
        logException(e);  
        return ResponseEntity.wrapMessageResponseEntity(HttpStatus.SERVICE_UNAVAILABLE.getReasonPhrase());  
    }  
  
    @ResponseStatus(HttpStatus.INTERNAL_SERVER_ERROR)  
    @ExceptionHandler(Exception.class)  
    public ResponseEntity handleException(Exception e) {  
        logException(e);  
        return ResponseEntity.wrapMessageResponseEntity(HttpStatus.INTERNAL_SERVER_ERROR.getReasonPhrase());  
    }  
  
    private void logException(Exception e) {  
        log.error(e.getMessage(), e);  
    }  
}
```

## 异常处理匹配优先级

有多个全局异常处理类，可使用`@Order`指定优先级。同一个全局异常处理类，不同的异常处理方法，优先匹配最精确的异常类型。

## Spring MVC默认异常处理类

当自定义异常处理类无法处理对应异常类型时，或者没有定义自定义异常处理类，将会由Spring MVC默认异常处理类`org.springframework.web.servlet.mvc.support.DefaultHandlerExceptionResolver`处理