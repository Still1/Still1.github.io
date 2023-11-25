---
title: MyBatis-Plus自定义InnerInterceptor使用
tags: [软件开发, Java, MyBatis-Plus]
---

## 代码实例

实现一个`InnerInterceptor`，对所有SQL进行拦截处理

```java
public class DeletedFilterInterceptor implements InnerInterceptor {
    @Override
    @SneakyThrows
    // 拦截select查询语句，对语句作处理
    public void beforeQuery(Executor executor, MappedStatement ms, Object parameter, RowBounds rowBounds, ResultHandler resultHandler, BoundSql boundSql) {
        String sql = boundSql.getSql();
        Select select = (Select) CCJSqlParserUtil.parse(sql);
        PlainSelect plainSelect = (PlainSelect) select.getSelectBody();
        Expression whereCondition = plainSelect.getWhere();

        Expression newWhereCondition = addDeletedCondition(whereCondition, select);
        plainSelect.setWhere(newWhereCondition);

        setBoundSql(boundSql, plainSelect.toString());
    }


    @Override
    @SneakyThrows
    // 重写这个方法无效，反射不起效果
    public void beforeUpdate(Executor executor, MappedStatement ms, Object parameter) {
        SqlCommandType sqlCommandType = ms.getSqlCommandType();
        if (sqlCommandType == SqlCommandType.UPDATE) {
            BoundSql boundSql = ms.getBoundSql(parameter);
            String sql = boundSql.getSql();
            Update update = (Update) CCJSqlParserUtil.parse(sql);
            Expression whereCondition = update.getWhere();

            Expression newWhereCondition = addDeletedCondition(whereCondition, update);
            update.setWhere(newWhereCondition);

            setBoundSql(boundSql, update.toString());
        }
    }

    @Override
    @SneakyThrows
    // 拦截update语句，对语句作处理
    public void beforePrepare(StatementHandler sh, Connection connection, Integer transactionTimeout) {
        BoundSql boundSql = sh.getBoundSql();
        MetaObject metaObject = MetaObject.forObject(sh, SystemMetaObject.DEFAULT_OBJECT_FACTORY, SystemMetaObject.DEFAULT_OBJECT_WRAPPER_FACTORY,  new DefaultReflectorFactory());
        MappedStatement ms = (MappedStatement) metaObject.getValue("delegate.mappedStatement");
        SqlCommandType sqlCommandType = ms.getSqlCommandType();
        if (sqlCommandType == SqlCommandType.UPDATE) {
            String sql = boundSql.getSql();
            Update update = (Update) CCJSqlParserUtil.parse(sql);
            Expression whereCondition = update.getWhere();

            Expression newWhereCondition = addDeletedCondition(whereCondition, update);
            update.setWhere(newWhereCondition);

            setBoundSql(boundSql, update.toString());
        }
    }

    // 往查询语句后面追加查询条件
    private Expression addDeletedCondition(Expression whereCondition, Statement statement) {
        TablesNamesFinder tablesNamesFinder = new TablesNamesFinder();
        List<String> tableList = tablesNamesFinder.getTableList(statement);

        IsBooleanExpression isBooleanExpression = new IsBooleanExpression();
        isBooleanExpression.setIsTrue(false);

        for (String table : tableList) {
            isBooleanExpression.setLeftExpression(new Column(new Table(table), DmsConstant.DELETED_COLUMN_NAME));
            if (whereCondition != null) {
                whereCondition = new AndExpression(whereCondition, isBooleanExpression);
            } else {
                whereCondition = isBooleanExpression;
            }
        }
        return whereCondition;
    }

    // 通过反射修改boundSql的sql属性，达到修改要执行的SQL语句的目的
    @SneakyThrows
    private void setBoundSql(BoundSql boundSql, String newSql) {
        Field sqlField = ReflectionUtils.findField(boundSql.getClass(), "sql");
        if (sqlField != null) {
            sqlField.setAccessible(true);
            sqlField.set(boundSql, newSql);
        }
    }
}
```

添加自定义的`InnerInterceptor`

```java
@Configuration  
public class MyBatisPlusConfig {  
    @Bean  
    public MybatisPlusInterceptor mybatisPlusInterceptor() {  
        // MybatisPlusInterceptor底层是MyBatis的Interceptor
        MybatisPlusInterceptor interceptor = new MybatisPlusInterceptor();  
  
        PaginationInnerInterceptor paginationInnerInterceptor = new PaginationInnerInterceptor();  
        paginationInnerInterceptor.setDbType(DbType.POSTGRE_SQL);  
        interceptor.addInnerInterceptor(paginationInnerInterceptor);  

        // 添加自定义的InnerInterceptor
        DeletedFilterInterceptor deletedFilterInterceptor = new DeletedFilterInterceptor();  
        interceptor.addInnerInterceptor(deletedFilterInterceptor);  
  
        CurrentUserOrganizationFilterInterceptor currentUserOrganizationFilterInterceptor = new CurrentUserOrganizationFilterInterceptor();  
        interceptor.addInnerInterceptor(currentUserOrganizationFilterInterceptor);  
        return interceptor;  
    }  
}
```