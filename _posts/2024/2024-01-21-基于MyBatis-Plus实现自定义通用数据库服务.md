---
title: 基于MyBatis-Plus实现自定义通用数据库服务
tags: [软件开发, Java, MyBatis-Plus]
---

## 实体

公用实体类
```java
package com.oc.odm.entity;  
  
import com.baomidou.mybatisplus.annotation.IdType;  
import com.baomidou.mybatisplus.annotation.TableId;  
import lombok.Data;  
  
import java.io.Serializable;  
import java.time.LocalDateTime;  
  
@Data  
public class OdmCommonEntity implements Serializable {  
    /**  
     * ID     
     */    
    @TableId(value = "id", type = IdType.AUTO)  
    private Integer id;  
    /**  
     * 创建者  
     */  
    private Integer createdBy;  
    /**  
     * 创建时间  
     */  
    private LocalDateTime createdTime;  
    /**  
     * 更新者  
     */  
    private Integer updatedBy;  
    /**  
     * 更新时间  
     */  
    private LocalDateTime updatedTime;  
    /**  
     * 是否删除  
     */  
    private Boolean deleted;  
}
```

具体的实体类，继承公用实体类，再作扩展
```java
package com.oc.odm.entity;  
  
import com.baomidou.mybatisplus.annotation.TableName;  
import com.oc.odm.config.permission.OrganizationIdOwner;  
import lombok.Data;  
import lombok.EqualsAndHashCode;  
  
import java.io.Serializable;  
  
@TableName("t_dict")  
@Data  
@EqualsAndHashCode(callSuper = false)  
public class Dict extends OdmCommonEntity implements Serializable {  
    /**  
     * 字典类型编码  
     */  
    private String code;  
  
    /**  
     * 字典类型名称  
     */  
    private String name;  
  
    /**  
     * 是否启用  
     */  
    private Boolean enabled;  
  
    /**  
     * 组织id  
     */  
    private Integer organizationId;  
}
```

## 查询条件

公用查询条件类
```java
package com.oc.odm.dto.condition;  
  
import lombok.Data;  
  
/**  
 * 通用查询条件DTO  
 */
@Data  
public class OdmCommonCondition {  
    /**  
     * 一页的数量  
     */  
    private Integer pageSize;  
  
    /**  
     * 页数  
     */  
    private Integer pageIndex;  
  
    /**  
     * 排序属性名  
     */  
    private String orderFieldName;  
  
    /**  
     * 是否以正序排序  
     */  
    private Boolean ascend;  
}
```

查询条件操作注解，用于指定每个查询条件的操作类型
```java
package com.oc.odm.dto.condition;  
  
import java.lang.annotation.*;  
  
@Target(ElementType.FIELD)  
@Retention(RetentionPolicy.RUNTIME)  
@Documented  
public @interface ConditionOperator {  
    ConditionOperatorType value() default ConditionOperatorType.EQUAL;  
}
```

查询条件操作类型枚举类
```java
package com.oc.odm.dto.condition;  
  
import com.baomidou.mybatisplus.core.conditions.interfaces.Compare;  
import lombok.Getter;  
import org.springframework.util.ReflectionUtils;  
  
import java.lang.reflect.Method;  
  
@Getter  
public enum ConditionOperatorType {  
    EQUAL(ReflectionUtils.findMethod(Compare.class, "eq", Object.class, Object.class)),  
    LIKE(ReflectionUtils.findMethod(Compare.class, "like", Object.class, Object.class));  
  
    private final Method queryWrapperMethod;  
  
    ConditionOperatorType(Method queryWrapperMethod) {  
        this.queryWrapperMethod = queryWrapperMethod;  
    }  
}
```

具体的查询条件类，继承公用查询条件类，再作扩展
```java
package com.oc.odm.dto.condition;  
  
import lombok.Data;  
import lombok.EqualsAndHashCode;  
  
/**  
 * 字典类型查询条件DTO  
 */
@Data  
@EqualsAndHashCode(callSuper = false)  
public class DictCondition extends OdmCommonCondition {  
    /**  
     * 字典类型名称  
     */  
    @ConditionOperator(ConditionOperatorType.LIKE)  
    private String name;  
    /**  
     * 字典类型编码  
     */  
    @ConditionOperator(ConditionOperatorType.EQUAL)  
    private String code;  
}
```

查询条件转换器
```java
package com.oc.odm.utils;  
  
import com.baomidou.mybatisplus.core.conditions.query.QueryWrapper;  
import com.baomidou.mybatisplus.core.toolkit.Wrappers;  
import com.baomidou.mybatisplus.extension.plugins.pagination.Page;  
import com.oc.odm.dto.condition.OdmCommonCondition;  
import com.oc.odm.dto.condition.ConditionOperator;  
import lombok.SneakyThrows;  
import org.apache.commons.lang3.StringUtils;  
import org.springframework.util.ReflectionUtils;  
  
import java.lang.reflect.Field;  
import java.lang.reflect.Method;  
  
/**  
 * 查询条件转换器  
 */  
public class ConditionConverter {  
  
    public static final String GETTER_METHOD_PREFIX = "get";  
  
    /**  
     * 查询条件对象转换为QueryWrapper  
     *     
     * @param odmCommonCondition 查询条件对象  
     * @return 转换后的QueryWrapper对象  
     * @param <T> QueryWrapper关联的实体类泛型  
     */  
    public static <T> QueryWrapper<T> convertToQueryWrapper(OdmCommonCondition odmCommonCondition) {  
        QueryWrapper<T> queryWrapper = Wrappers.query();  
        handleWhere(odmCommonCondition, queryWrapper);  
  
        String orderFieldName = odmCommonCondition.getOrderFieldName();  
        Boolean ascend = odmCommonCondition.getAscend();  
        if (StringUtils.isNotBlank(orderFieldName) && ascend != null) {  
            handleOrder(orderFieldName, ascend, queryWrapper);  
        }  
  
        return queryWrapper;  
    }  
  
    /**  
     * 处理where查询条件  
     * 查询条件对象值不为空的属性加入QueryWrapper的查询条件，多个查询条件之间的关系为and  
     *     
     * @param odmCommonCondition 查询条件对象  
     * @param queryWrapper 要处理的QueryWrapper对象  
     */  
    @SneakyThrows  
    private static void handleWhere(OdmCommonCondition odmCommonCondition, QueryWrapper<?> queryWrapper) {  
        Class<? extends OdmCommonCondition> odmCommonConditionClass = odmCommonCondition.getClass();  
        Method[] declaredMethods = ReflectionUtils.getDeclaredMethods(odmCommonConditionClass);  
        for (Method method : declaredMethods) {  
            String methodName = method.getName();  
            if (methodName.startsWith(GETTER_METHOD_PREFIX)) {  
                Object value = method.invoke(odmCommonCondition);  
                if (value != null) {  
                    String fieldName = OdmStringUtils.lowerCaseFirstCharacter(methodName.substring(3));  
                    String columnName = OdmStringUtils.camelCaseToSnakeCase(fieldName);  
                    Field field = ReflectionUtils.findField(odmCommonConditionClass, fieldName);  
                    if (field != null) {  
                        ConditionOperator conditionOperator = field.getAnnotation(ConditionOperator.class);  
                        if (conditionOperator == null) {  
                            queryWrapper.eq(columnName, value);  
                        } else {  
                            conditionOperator.value().getQueryWrapperMethod().invoke(queryWrapper, columnName, value);  
                        }  
                    }  
                }  
            }  
        }  
    }  
  
    /**  
     * 处理排序  
     *  
     * @param orderFieldName 排序字段  
     * @param ascend 是否正序排序  
     * @param queryWrapper 要处理的QueryWrapper对象  
     */  
    private static void handleOrder(String orderFieldName, boolean ascend, QueryWrapper<?> queryWrapper) {  
        String orderColumnName = OdmStringUtils.camelCaseToSnakeCase(orderFieldName);  
        if (ascend) {  
            queryWrapper.orderByAsc(orderColumnName);  
        } else {  
            queryWrapper.orderByDesc(orderColumnName);  
        }  
    }  
  
  
    /**  
     * 查询条件转换为Page  
     *     
     * @param odmCommonCondition 通用查询条件  
     * @return 分页处理Page对象  
     * @param <T> Page对象关联的实体类泛型  
     */  
    public static <T> Page<T> convertToPage(OdmCommonCondition odmCommonCondition) {  
        Integer pageIndex = odmCommonCondition.getPageIndex();  
        Integer pageSize = odmCommonCondition.getPageSize();  
        if (pageIndex != null && pageSize != null) {  
            Page<T> page = new Page<>();  
            page.setSize(pageSize);  
            page.setCurrent(pageIndex);  
            return page;  
        } else {  
            return null;  
        }  
    }  
}
```

## Mapper

```java
package com.oc.odm.mapper;  
  
import com.oc.odm.entity.Dict;  
import com.baomidou.mybatisplus.core.mapper.BaseMapper;  
import org.springframework.stereotype.Repository;  
  
@Repository  
public interface DictMapper extends BaseMapper<Dict> {  
}
```

## Manager

Manager层主要负责处理缓存

```java
package com.oc.odm.manager;  
  
import com.baomidou.mybatisplus.core.mapper.BaseMapper;  
import com.oc.odm.dto.condition.OdmCommonCondition;  
import com.oc.odm.entity.OdmCommonEntity;  
  
import java.util.Collection;  
import java.util.List;  
  
public interface OdmCommonManager<T extends OdmCommonEntity, U extends OdmCommonCondition, M extends BaseMapper<T>> {  
    T getById(Integer id);  
  
    List<T> getList(U condition);  
  
    void save(T entity);  
  
    void updateById(T entity);  
  
    String getEntityName();  
  
    M getBaseMapper();  
  
    T createEntityObject();  
  
    void saveBatch(Collection<T> entityList, int batchSize);  
  
    void updateBatchById(Collection<T> entityList, int batchSize);  
}
```


```java
package com.oc.odm.manager;  
  
import com.baomidou.mybatisplus.core.conditions.query.QueryWrapper;  
import com.baomidou.mybatisplus.core.enums.SqlMethod;  
import com.baomidou.mybatisplus.core.mapper.BaseMapper;  
import com.baomidou.mybatisplus.core.toolkit.Constants;  
import com.baomidou.mybatisplus.extension.plugins.pagination.Page;  
import com.baomidou.mybatisplus.extension.toolkit.SqlHelper;  
import com.oc.odm.dto.condition.OdmCommonCondition;  
import com.oc.odm.entity.OdmCommonEntity;  
import com.oc.odm.utils.ConditionConverter;  
import com.oc.odm.utils.OdmStringUtils;  
import lombok.SneakyThrows;  
import org.apache.ibatis.binding.MapperMethod;  
import org.apache.ibatis.logging.Log;  
import org.apache.ibatis.logging.LogFactory;  
import org.apache.ibatis.session.SqlSession;  
import org.springframework.beans.factory.annotation.Autowired;  
import org.springframework.cache.annotation.CacheEvict;  
import org.springframework.cache.annotation.Cacheable;  
  
import java.lang.reflect.ParameterizedType;  
import java.lang.reflect.Type;  
import java.util.Collection;  
import java.util.List;  
import java.util.function.BiConsumer;  
  
public abstract class AbstractOdmCommonManager<T extends OdmCommonEntity, U extends OdmCommonCondition, M extends BaseMapper<T>> implements OdmCommonManager<T, U, M> {  
    @Autowired(required = false)  
    private M baseMapper;  
  
    private final Log log = LogFactory.getLog(getClass());  
  
    @Override  
    @Cacheable(cacheResolver = "entityCacheNamesCacheResolver")  
    public T getById(Integer id) {  
        return baseMapper.selectById(id);  
    }  
  
    @Override  
    public List<T> getList(U condition) {  
        QueryWrapper<T> queryWrapper = ConditionConverter.convertToQueryWrapper(condition);  
        Page<T> page = ConditionConverter.convertToPage(condition);  
        if (page == null) {  
            return baseMapper.selectList(queryWrapper);  
        } else {  
            return baseMapper.selectPage(page, queryWrapper).getRecords();  
        }  
    }  
  
    @Override  
    @CacheEvict(cacheResolver = "saveCacheEvictCacheResolver", allEntries = true)  
    public void save(T entity) {  
        baseMapper.insert(entity);  
    }  
  
    @Override  
    @CacheEvict(cacheResolver = "updateOrDeleteCacheEvictCacheResolver", allEntries = true)  
    public void updateById(T entity) {  
        baseMapper.updateById(entity);  
    }  
  
    @Override  
    public String getEntityName() {  
        Type entityType = getEntityType();  
        if (entityType != null) {  
            String typeName = entityType.getTypeName();  
            String[] split = typeName.split("\\.");  
            return OdmStringUtils.lowerCaseFirstCharacter(split[split.length - 1]);  
        }  
        return null;  
    }  
  
    @Override  
    public M getBaseMapper() {  
        return baseMapper;  
    }  
  
    @Override  
    @CacheEvict(cacheResolver = "saveCacheEvictCacheResolver", allEntries = true)  
    public void saveBatch(Collection<T> entityCollection, int batchSize) {  
        String sqlStatement = getSqlStatement(SqlMethod.INSERT_ONE);  
        executeBatch(entityCollection, batchSize, (sqlSession, entity) -> sqlSession.insert(sqlStatement, entity));  
    }  
  
    @Override  
    @CacheEvict(cacheResolver = "updateOrDeleteCacheEvictCacheResolver", allEntries = true)  
    public void updateBatchById(Collection<T> entityCollection, int batchSize) {  
        String sqlStatement = getSqlStatement(SqlMethod.UPDATE_BY_ID);  
        executeBatch(entityCollection, batchSize, (sqlSession, entity) -> {  
            MapperMethod.ParamMap<T> param = new MapperMethod.ParamMap<>();  
            param.put(Constants.ENTITY, entity);  
            sqlSession.update(sqlStatement, param);  
        });  
    }  
  
    @SneakyThrows  
    @SuppressWarnings("unchecked")  
    public T createEntityObject() {  
        Type entityType = getEntityType();  
        if (entityType != null) {  
            Class<T> entityClass = (Class<T>) entityType;  
            return entityClass.newInstance();  
        }  
        return null;  
    }  
  
    private Type getEntityType() {  
        return getParameterizedType(0);  
    }  
  
    private Type getMapperType() {  
        return getParameterizedType(2);  
    }  
  
    private Type getParameterizedType(int index) {  
        Type genericSuperclass = getClass().getGenericSuperclass();  
        if (genericSuperclass instanceof ParameterizedType) {  
            ParameterizedType parameterizedType = (ParameterizedType) genericSuperclass;  
            Type[] actualTypeArguments = parameterizedType.getActualTypeArguments();  
            if (actualTypeArguments.length > index) {  
                return actualTypeArguments[index];  
            }  
        }  
        return null;  
    }  
  
    private String getSqlStatement(SqlMethod sqlMethod) {  
        return SqlHelper.getSqlStatement((Class<?>) getMapperType(), sqlMethod);  
    }  
  
    private <E> void executeBatch(Collection<E> list, int batchSize, BiConsumer<SqlSession, E> consumer) {  
        SqlHelper.executeBatch((Class<?>) getEntityType(), this.log, list, batchSize, consumer);  
    }  
}
```

```java
package com.oc.odm.manager.impl;  
  
import com.oc.odm.dto.condition.DictCondition;  
import com.oc.odm.entity.Dict;  
import com.oc.odm.manager.AbstractOdmCommonManager;  
import com.oc.odm.manager.DictManager;  
import com.oc.odm.mapper.DictMapper;  
import org.springframework.stereotype.Component;  
  
@Component  
public class DictManagerImpl extends AbstractOdmCommonManager<Dict, DictCondition, DictMapper> implements DictManager {  
}
```

## Service

Service层主要处理事务
```java
package com.oc.odm.service;  
  
import com.oc.odm.dto.condition.OdmCommonCondition;  
import com.oc.odm.entity.OdmCommonEntity;  
  
import java.util.Collection;  
import java.util.List;  
  
public interface OdmCommonService<T extends OdmCommonEntity, U extends OdmCommonCondition> {  
    T getById(Integer id);  
  
    List<T> getList(U condition);  
  
    void save(T entity);  
  
    void updateById(T entity);  
  
    void deleteById(Integer id);  
  
    void saveOrUpdate(T entity);  
  
    void updateBatchById(Collection<T> entityCollection);  
  
    void saveBatch(Collection<T> entityCollection);  
  
    void deleteBatch(Collection<Integer> idCollection);  
  
    void saveOrUpdateBatch(Collection<T> entityCollection);  
}
```

```java
package com.oc.odm.service;  
  
import com.baomidou.mybatisplus.core.mapper.BaseMapper;  
import com.oc.odm.dto.condition.OdmCommonCondition;  
import com.oc.odm.entity.OdmCommonEntity;  
import com.oc.odm.manager.OdmCommonManager;  
import com.oc.odm.utils.OdmSecurityUtils;  
import lombok.SneakyThrows;  
import org.springframework.beans.factory.annotation.Autowired;  
import org.springframework.transaction.annotation.Transactional;  
  
import java.time.LocalDateTime;  
import java.util.Collection;  
import java.util.List;  
  
public abstract class AbstractOdmCommonService<T extends OdmCommonEntity, U extends OdmCommonCondition, M extends BaseMapper<T>, N extends OdmCommonManager<T, U, M>> implements OdmCommonService<T, U> {  
  
    private static final int BATCH_SIZE = 1000;  
  
    @Autowired(required = false)  
    private N manager;  
  
    @Override  
    public T getById(Integer id) {  
        return manager.getById(id);  
    }  
  
    @Override  
    public List<T> getList(U condition) {  
        return manager.getList(condition);  
    }  
  
    @Override  
    public void save(T entity) {  
        Integer currentUserId = OdmSecurityUtils.getCurrentUserId();  
        entity.setCreatedBy(currentUserId);  
        entity.setUpdatedBy(currentUserId);  
        entity.setCreatedTime(LocalDateTime.now());  
        entity.setUpdatedTime(LocalDateTime.now());  
        manager.save(entity);  
    }  
  
    @Override  
    public void updateById(T entity) {  
        Integer currentUserId = OdmSecurityUtils.getCurrentUserId();  
        entity.setUpdatedBy(currentUserId);  
        entity.setUpdatedTime(LocalDateTime.now());  
        manager.updateById(entity);  
    }  
  
    @Override  
    public void deleteById(Integer id) {  
        T entityObject = manager.createEntityObject();  
        entityObject.setId(id);  
        entityObject.setDeleted(true);  
        Integer currentUserId = OdmSecurityUtils.getCurrentUserId();  
        entityObject.setUpdatedBy(currentUserId);  
        entityObject.setUpdatedTime(LocalDateTime.now());  
        manager.updateById(entityObject);  
    }  
  
    @Override  
    @SneakyThrows    
    public void saveOrUpdate(T entity) {  
        Integer id =  entity.getId();  
        if (id != null) {  
            updateById(entity);  
        } else {  
            save(entity);  
        }  
    }  
  
    @Override  
    @Transactional    
    public void updateBatchById(Collection<T> entityCollection) {  
        for (T t : entityCollection) {  
            Integer currentUserId = OdmSecurityUtils.getCurrentUserId();  
            t.setUpdatedBy(currentUserId);  
            t.setUpdatedTime(LocalDateTime.now());  
        }  
  
        manager.updateBatchById(entityCollection, BATCH_SIZE);  
    }  
  
    @Override  
    @Transactional    
    public void saveBatch(Collection<T> entityCollection) {  
        for (T t : entityCollection) {  
            Integer currentUserId = OdmSecurityUtils.getCurrentUserId();  
            t.setCreatedBy(currentUserId);  
            t.setUpdatedBy(currentUserId);  
            t.setCreatedTime(LocalDateTime.now());  
            t.setUpdatedTime(LocalDateTime.now());  
        }  
  
        manager.saveBatch(entityCollection, BATCH_SIZE);  
    }  
  
    @Override  
    @Transactional    
    public void deleteBatch(Collection<Integer> idCollection) {  
        for (Integer id : idCollection) {  
            deleteById(id);  
        }  
    }  
  
    @Override  
    @Transactional    
    public void saveOrUpdateBatch(Collection<T> entityCollection) {  
        for (T t : entityCollection) {  
            saveOrUpdate(t);  
        }  
    }  
  
    protected N getManager() {  
        return manager;  
    }  
  
    protected M getBaseMapper() {  
        return getManager().getBaseMapper();  
    }  
}
```

```java
package com.oc.odm.service.impl;  
  
import com.oc.odm.dto.condition.DictCondition;  
import com.oc.odm.entity.Dict;  
import com.oc.odm.manager.DictManager;  
import com.oc.odm.mapper.DictMapper;  
import com.oc.odm.service.AbstractOdmCommonService;  
import com.oc.odm.service.DictService;  
import org.springframework.stereotype.Service;  
  
@Service  
public class DictServiceImpl extends AbstractOdmCommonService<Dict, DictCondition, DictMapper, DictManager> implements DictService {  
}
```