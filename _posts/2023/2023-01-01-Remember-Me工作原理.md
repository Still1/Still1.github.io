---
title: Remember-Me工作原理
tags: [网络安全, 概念原理]
---

## 工作流程

### 首次登录

![](https://oliver-blog.oss-cn-shenzhen.aliyuncs.com/20230226115255.png)

在首次登录时，客户端发送的数据中提示开启remember-me功能。服务器接收到登录请求数据后，按正常流程先根据用户名和密码验证用户的有效性，验证通过后，生成remember-me token。remember-me token中包含用户名，token有效期和用于验证token有效性的摘要信息。最后服务器在对客户端的响应中添加这个remember-me token

### session失效后使用remember-me token登录

<img src="https://oliver-blog.oss-cn-shenzhen.aliyuncs.com/20230226115826.png" width="700px" />

session失效后，客户端请求服务器资源时仍然会通过cookie带上未过期的remember-me token。服务器接收到这个token后，先验证token的合法性，包括其是否在有效期内，以及重新计算摘要信息以比对token中的摘要信息，确认token未被篡改。验证通过后，服务器为此用户登录，被返回客户端请求的资源内容