---
title: DockerFile指南
tags: 
  - Docker
modify_date: 2022-09-08
---

## 关键字 

FROM 指定基础镜像

<!--more--> 

RUN 构建镜像时执行

```dockerfile
RUN ["./test.php", "dev", "offline"]
RUN ./test.php dev offline
```

EXPOSE 暴露端口

WORKDIR 指定初始工作目录

ENV 设置环境变量

```dockerfile
ENV CATALINA_HOME /usr/local/tomcat
```

ADD 将宿主机目录下的文件复制到镜像中并自动处理URL和解压

COPY 复制宿主机的文件和目录，类似ADD

VOLUME 指定容器卷

CMD 容器启动后默认运行的命令，可以被`docker run`命令的参数覆盖

```dockerfile
CMD ["catalina.sh", "run"]
```

ENTRYPOINT 容器启动后默认运行的命令，优先级比CMD高。ENTRYPOINT与CMD一起使用，则CMD只控制参数

```dockerfile
ENTRYPOINT ["nginx", "-c"]
CMD ["/etc/nginx/nginx.conf"]
```

## 例子

文件名Dockerfile

```dockerfile
FROM centos

ENV MYPATH /usr/local
WORKDIR $MYPATH

RUN yum -y install vim
RUN yum -y install net-tools
RUN yum -y install glibc.i686

RUN mkdir /usr/local/java
ADD jdk-8u171-linux-x64.tar.gz /usr/local/java

ENV JAVA_HOME /usr/local/java/jdk1.8.0_171
ENV JRE_HOME $JAVA_HOME/jre
ENV CLASSPATH $JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar:$JRE_HOME/lib:$CLASSPATH
ENV PATH $JAVA_HOME/bin:$PATH

EXPOSE 80

CMD /bin/bash
```

Spring Boot应用镜像例子
```dockerfile
FROM openjdk:11
RUN mkdir -p /opt/languageTrainer
COPY LanguageTrainer.jar /opt/languageTrainer/LanguageTrainer.jar
EXPOSE 8080
CMD ["nohup", "java", "-jar", "/opt/languageTrainer/LanguageTrainer.jar", "&"]
```

## 构建镜像

在存在Dockerfile的目录下执行

```shell
# 最后的点，表示Docker构建的上下文环境，同时也是COPY命令的相对路径
docker build -t centosjava8:1.5 .
```

```shell
docker build -f /opt/languageTrainer/Dockerfile -t "language-trainer" /opt/languageTrainer/
```



