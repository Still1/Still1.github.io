---
title: GitLab提交代码触发Jenkins构建
tags: [Jenkins, GitLab, CI&CD, 软件开发]
---

## 安装Jenkins插件

安装GitLab插件

![image-20220918102104731](https://oliver-blog.oss-cn-shenzhen.aliyuncs.com/20220918102105.png)



## 配置项目构建触发器

![image-20220918110903807](https://oliver-blog.oss-cn-shenzhen.aliyuncs.com/20220918110905.png)

![image-20220918110938846](https://oliver-blog.oss-cn-shenzhen.aliyuncs.com/20220918110949.png)



## GitLab出站请求配置

访问本地网络的Jenkins时需要配置

![image-20220918110150142](https://oliver-blog.oss-cn-shenzhen.aliyuncs.com/20220918110151.png)



## 项目Webhook配置

![image-20220918110706870](https://oliver-blog.oss-cn-shenzhen.aliyuncs.com/20220918110709.png)

非HTTPS取消启用SSL验证

![image-20220918110731205](https://oliver-blog.oss-cn-shenzhen.aliyuncs.com/20220918110732.png)