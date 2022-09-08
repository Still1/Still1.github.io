---
title: Docker下的GitLab安装
tags: 
  - GitLab
  - Docker
---

## 配置环境变量

```shell
export GITLAB_HOME=/srv/gitlab
```

<!--more-->

## 启动GitLab容器

```
docker run --detach \
  --hostname gitlab.oc.com \
  --publish 443:443 --publish 80:80 --publish 2222:22 \
  --name gitlab \
  --restart always \
  --volume $GITLAB_HOME/config:/etc/gitlab:Z \
  --volume $GITLAB_HOME/logs:/var/log/gitlab:Z \
  --volume $GITLAB_HOME/data:/var/opt/gitlab:Z \
  --shm-size 256m \
  registry.gitlab.cn/omnibus/gitlab-jh:latest
```

## 初始登录

获取初始密码

```
docker exec -it gitlab grep 'Password:' /etc/gitlab/initial_root_password
```

使用用户名root和初始密码登录

