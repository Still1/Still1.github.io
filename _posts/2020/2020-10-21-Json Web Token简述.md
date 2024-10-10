---
title: Java Web Token简述
tags: [网络安全, 概念原理]
---

## Json Web Token信息组成

Java Web Token信息由三部分组成，分别是

* Header，描述token的类型，摘要算法等，JSON格式
* Payload，描述token认证相关的一些主要信息，JSON格式
* Verify Signature，token的摘要信息，用于检查token的合法性

### 例子

Header

```json
{
  "alg": "HS256",
  "typ": "JWT"
}
```

Payload

```json
{
  "sub": "1234567890",
  "name": "John Doe",
  "iat": 1516239022
}
```

Verify Signature

这里通过伪代码表示，`HMACSHA256`表示摘要生成算法，`base64UrlEncode`表示对信息作base64编码的方法，`header`变量表示header部分的JSON格式信息，`payload`变量表示payload部分的JSON格式信息，`secret`变量表示摘要生成算法的密钥

```
HMACSHA256(
  base64UrlEncode(header) + "." +
  base64UrlEncode(payload),
  secret
)
```

## Json Web Token的生成

依次对header、payload、verify signature三部分的信息作base64编码，然后通过`.`字符拼接起来，即可生成token。根据上文例子的信息，可以生成以下token

```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiaWF0IjoxNTE2MjM5MDIyfQ.XbPfbIHMI6arZ3Y922BhjWgQzWXcXNrz0ogtVhfEd2o
```

通过伪代码来表示生成逻辑

```
base64Header = base64UrlEncode(header);
base64Payload = base64UrlEncode(payload);
signature = HMACSHA256(base64Header + "." + base64Payload, secret);
base64Signature = base64UrlEncode(signature);

token = base64Header + "." + base64Payload + "." + base64Signature
```

## Json Web Token的验证

通过重新计算token的摘要信息，与token中的提供摘要信息进行对比，即可验证token是否曾被篡改