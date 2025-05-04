---
title: 解决Java URL中文参数 后台request接收乱码的问题
tags: [Java, 问题解决]
---

## 解决方法

1.可以考虑中间件编码配置
例如tomcat server.xml 的配置
![](https://oliver-blog.oss-cn-shenzhen.aliyuncs.com/20240405184511.png)

eclipse的tomcat server 需要删掉再new一个，配置才生效

这个设置是影响URI中的字符编码，一般影响get请求的query string
tomcat8后默认UTF-8，不需要另外设置


2.加上Spring的字符集filter
![](https://oliver-blog.oss-cn-shenzhen.aliyuncs.com/20240405184543.png)

需确保该filter在最前面执行
注意Spring Security的filter一般会在最前面
Spring Security的CSRF检查会提早访问HttpServletRequest的参数，导致CharacterEncodingFilter失效

```java
public class SecurityWebApplicationInitializer extends AbstractSecurityWebApplicationInitializer {
    @Override
    protected void beforeSpringSecurityFilterChain(ServletContext servletContext) {
        insertFilters(servletContext, new CharacterEncodingFilter(StandardCharsets.UTF_8.name()));
    }
}
```


3.文件编码类型与tomcat的file.encoding参数不对应
JVM参数加上 -Dfile.encoding=UTF-8
IDE中可以在 VM arguement或VM options中添加
![](https://oliver-blog.oss-cn-shenzhen.aliyuncs.com/20240405184620.png)

tomcat可以在 CATALINA_OPTS或JAVA_OPTS环境变量中添加，推荐CATALINA_OPTS

tomcat8.5后不需要加，应为默认