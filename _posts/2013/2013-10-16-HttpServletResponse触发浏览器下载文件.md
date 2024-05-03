---
title: HttpServletResponse触发浏览器下载文件
tags: [Servlet, Java]
---

## Java代码例子

```java
public static void download(HttpServletResponse response, String fileName, byte[] content) throws Exception {
    OutputStream toClient = new BufferedOutputStream(response.getOutputStream());
    response.reset();
    response.setContentType("application/octet-stream");
    response.setBufferSize(1024);
    response.setHeader("Content-Disposition", "attachment; filename=" + new String(fileName.getBytes("gb2312"),"iso8859-1"));
    response.addHeader("Content-Length", String.valueOf(content.length));
    toClient.write(content);
    toClient.flush();
    toClient.close();
}
```

MIME类型`application/octet-stream`可以表示任意二进制数据
HTTP头`Content-Disposition`加上`attachement`可以触发浏览器下载
注意：使用AJAX方式，无法触发浏览器下载