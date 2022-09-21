---
title: Jenkins自动化部署Maven项目搭建
tags: 
  - Jenkins
---

## Docker安装并启动Jenkins

<!--more-->

```
docker pull jenkinsci/blueocean

docker run \
  --name jenkins \
  -u root \
  -d \
  -p 80:8080 \
  -p 50000:50000 \
  -v jenkins-data:/var/jenkins_home \
  -v /var/run/docker.sock:/var/run/docker.sock \
  --restart always \
  jenkinsci/blueocean
```



## 访问Jenkins

[http://192.168.88.9:8080/](http://192.168.88.9:8080/)

![image-20220906043941645](https://oliver-blog.oss-cn-shenzhen.aliyuncs.com/20220906044141.png)

按步骤做初始化操作

初始密码挂载路径

`/var/lib/docker/volumes/jenkins-data/_data/secrets/initialAdminPassword`



## 安装插件

第一次访问Jenkins时，选择安装推荐使用的插件

再安装以下插件

* Maven Intergration
* Publish Over SSH

### Jenkins插件更新网络问题解决

点击插件更新提示的Correct按钮或点击下方的Manage Plugins

![image-20220906053852405](https://oliver-blog.oss-cn-shenzhen.aliyuncs.com/20220906053853.png)

点击Advanced页签

![image-20220906054139887](https://oliver-blog.oss-cn-shenzhen.aliyuncs.com/20220906054141.png)

配置Update Site，修改为

`http://mirror.esuni.jp/jenkins/updates/update-center.json`



## 配置Jenkins

全局工具配置

### 配置JDK

![image-20220906061738683](https://oliver-blog.oss-cn-shenzhen.aliyuncs.com/20220922061337.png)

`/opt/java/openjdk`

### 配置Git

![image-20220906062109405](https://oliver-blog.oss-cn-shenzhen.aliyuncs.com/20220922061350.png)

`/usr/bin/git`

### 安装Maven

![image-20220915133333909](https://oliver-blog.oss-cn-shenzhen.aliyuncs.com/20220915133401.png)



## SSH远程应用服务器配置

![image-20220916072423833](C:\Users\Oliver\Desktop\upload\image-20220916072423833.png)



## 构建Maven项目

![image-20220915145758227](https://oliver-blog.oss-cn-shenzhen.aliyuncs.com/20220915145800.png)

### 配置源码管理

![image-20220915180355495](https://oliver-blog.oss-cn-shenzhen.aliyuncs.com/20220915180358.png)

### 配置构建前操作

清理之前构建的数据

![image-20220917165909988](https://oliver-blog.oss-cn-shenzhen.aliyuncs.com/20220917165935.png)

```
rm -rf /opt/languageTrainer
mkdir -p /opt/languageTrainer
```

### 配置构建

![image-20220917170059580](https://oliver-blog.oss-cn-shenzhen.aliyuncs.com/20220917170100.png)

### 配置构建后操作

#### 传输应用jar文件，并修改文件名

![image-20220918100426987](https://oliver-blog.oss-cn-shenzhen.aliyuncs.com/20220918100458.png)

```shell
mv /opt/languageTrainer/*.jar /opt/languageTrainer/LanguageTrainer.jar
```

#### 传输DockerFile

![image-20220918100448082](https://oliver-blog.oss-cn-shenzhen.aliyuncs.com/20220918100515.png)

#### 传输shell脚本

![image-20220918100636376](https://oliver-blog.oss-cn-shenzhen.aliyuncs.com/20220918100643.png)

```
chmod +x /opt/languageTrainer/*.sh
/opt/languageTrainer/clean.sh
/opt/languageTrainer/run.sh
```

clean.sh

```shell
#!/bin/bash
sudo echo "-----Cleaning the container of Language Trainer begins.-----"
containerId=$(sudo docker ps | grep -w language-trainer | head -1 | awk '{print $1}')
if [ "$containerId" ]; then
  sudo echo "The container of Language Trainer exists, and it is running now."
  sudo echo "Stop the container. Container Id:$containerId"
  sudo docker update --restart=no "$containerId"
  sudo docker stop "$containerId"
  sudo echo "Remove the container. Container Id:$containerId"
  sudo docker rm "$containerId"
else
  containerId=$(sudo docker ps -a | grep -w language-trainer | head -1 | awk '{print $1}')
  if [ "$containerId" ]; then
    sudo echo "The container of Language Trainer exists, but it is not running now."
    sudo echo "Remove the container. Container Id:$containerId"
    sudo docker rm "$containerId"
  else
    sudo echo "The container of Language Trainer doesn't exists."
  fi
fi
sudo echo "-----Cleaning the container of Language Trainer ends.-----"

sudo echo "-----Cleaning the image of Language Trainer begins.-----"
imageId=$(sudo docker images | grep -w language-trainer | head -1 | awk '{print $3}')
if [ "$imageId" ]; then
  sudo echo "The image of Language Trainer exists."
  sudo echo "Remove the image. Image Id:$imageId"
  sudo docker rmi "$imageId"
else
  sudo echo "The image of Language Trainer doesn't exists."
fi
sudo echo "-----Cleaning the image of Language Trainer ends.-----"
```

run.sh

```shell
#!/bin/bash
sudo docker build -f /opt/languageTrainer/Dockerfile -t "language-trainer" /opt/languageTrainer/
sudo docker run -p 8080:8080 --name "language-trainer" --restart always -d "language-trainer"
```

### 配置Jenkins最后清理工作

![image-20220918104307729](https://oliver-blog.oss-cn-shenzhen.aliyuncs.com/20220918104349.png)