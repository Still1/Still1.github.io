---
title: 使用InputStream读入字符串最佳实践
tags: [Java]
---

## 字节流，效率最高

```java
byte[] buffer = new byte[1024];
ByteArrayOutputStream byteArrayOutputStream = new ByteArrayOutputStream();
int length;
while ((length = inputStream.read(buffer)) != -1) {
    byteArrayOutputStream.write(buffer, 0, length);
}
return byteArrayOutputStream.toString(StandardCharsets.UTF_8.name());
```

## 字符流，效率稍次

```java
char[] buffer = new char[1024];
Reader reader = new InputStreamReader(inputStream, StandardCharsets.UTF_8.name());
StringBuilder stringBuilder = new StringBuilder();
int length;
while ((length = reader.read(buffer, 0, buffer.length)) != -1) {
    stringBuilder.append(buffer, 0, length);
}
return stringBuilder.toString();
```

## 调用类库，最简单

对长字符串效率不错

```java
String result = IOUtils.toString(inputStream, StandardCharsets.UTF_8.name());
```

