---
title: Spring Boot编程式事务
tags: [Spring Boot]
---

## 无需处理返回值

注入`TransactionTemplate`

```java
transactionTemplate.execute(new TransactionCallbackWithoutResult() {  
    @Override  
    protected void doInTransactionWithoutResult(@NotNull TransactionStatus status) {  
        if(!checkUserLoginLogExistence(localDate)) {  
            UserLoginStat userLoginStat = getUserLoginStat(userId);  
            Integer loginDays = userLoginStat.getLoginDays();  
            userLoginStat.setLoginDays(loginDays + 1);  

            LocalDate yesterday = localDate.minusDays(1);  
            Integer straightLoginDays = userLoginStat.getStraightLoginDays();  
            if(checkUserLoginLogExistence(yesterday)) {  
                userLoginStat.setStraightLoginDays(straightLoginDays + 1);  
            } else {  
                userLoginStat.setStraightLoginDays(1);  
            }  

            userLoginStatMapper.updateById(userLoginStat);  
        }  
        addLoginSuccessLog(userId, localDateTime);  
    }  
});
```