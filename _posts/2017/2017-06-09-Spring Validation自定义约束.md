---
title: Spring Validation自定义约束
tags: [Spring]
---

## 完整实践案例

### Maven依赖

```java
<dependency>  
   <groupId>org.springframework.boot</groupId>  
   <artifactId>spring-boot-starter-validation</artifactId>  
</dependency>
```

### 定义约束注解

```java
@Target(ElementType.FIELD)  
@Retention(RetentionPolicy.RUNTIME)  
@Constraint(validatedBy = CheckCorpusModeValidator.class)  
@Documented  
@Repeatable(CheckCorpusMode.CheckCorpusModes.class)  
public @interface CheckCorpusMode {  
    String message() default "corpus mode is not found";  
  
    Class<?>[] groups() default {};  
  
    Class<? extends Payload>[] payload() default {};  
  
    @Target(ElementType.FIELD)  
    @Retention(RetentionPolicy.RUNTIME)  
    @Documented  
    @interface CheckCorpusModes {  
        CheckCorpusMode[] value();  
    }  
}
```

### 定义约束验证器

```java
public class CheckCorpusModeValidator implements ConstraintValidator<CheckCorpusMode, String> {  
    @Override  
    public void initialize(CheckCorpusMode constraintAnnotation) {  
        ConstraintValidator.super.initialize(constraintAnnotation);  
    }  
  
    @Override  
    public boolean isValid(String value, ConstraintValidatorContext constraintValidatorContext) {  
        return CommonConstant.CorpusMode.checkIfExistsByModeName(value);  
    }  
}
```

### 使用约束注解

```java
@Data  
public class CompleteCorpusInDto {  
    @Min(1)  
    private Integer userId;  
    @Min(1)  
    private Integer corpusId;  
    @CheckCorpusMode  
    private String corpusModeName;  
}
```

### 开启验证参数

```java
@PostMapping("completeCorpus")  
public ResponseEntity completeCorpus(@RequestBody @Validated CompleteCorpusInDto completeCorpusInDto) {  
    Integer userId = completeCorpusInDto.getUserId();  
    Integer corpusId = completeCorpusInDto.getCorpusId();  
    String corpusModeName = completeCorpusInDto.getCorpusModeName();  
    CommonConstant.CorpusMode corpusMode = CommonConstant.CorpusMode.getModeByModeName(corpusModeName);  
  
    userService.completeCorpus(userId, corpusId, corpusMode);  
    return ResponseEntity.wrapSuccessMessageResponseEntity();  
}
```

## 参考文档

* [https://docs.jboss.org/hibernate/validator/6.2/reference/en-US/html_single/#validator-customconstraints-simple](https://docs.jboss.org/hibernate/validator/6.2/reference/en-US/html_single/#validator-customconstraints-simple)