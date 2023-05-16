---
title: Docker下的GitLab安装
tags: [Docker, 软件开发, GitLab]
---

## 配置临时环境变量

```shell
export GITLAB_HOME=/srv/gitlab
```

## 启动GitLab容器

```
docker run --detach \
  --hostname gitlab.oliverclio.com \
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



## SSH端口

### 修改本地提交密钥配置的端口

由于22端口在远程操作时被占用，GitLab的SSH端口映射到2222，在与GitLab建立SSH连接时，需要配置端口

打开`~/.ssh/config`文件，如果没有则新建

```
Host gitlab.oliverclio.com
Port 2222
```

### 修改GitLab的SSH默认端口

修改GitLab服务器`/srv/gitlab/config/gitlab.rb`配置文件，修改后重启Docker容器

```
gitlab_rails['gitlab_shell_ssh_port']= 2222
```



## 数据备份与恢复

docker方式启动，数据都在数据卷中，一般不需要备份

### 数据备份

```
docker exec -it gitlab /bin/bash
gitlab-rake gitlab:backup:create
```

备份文件 `usr/local/docker/gitlab/data/backups/gitlab_backup.tar`

### 数据恢复

```
gitlab-rake gitlab:backup:restore BACKUP=gitlab_backup
```



## GitLab无法访问Gravatar头像服务解决

修改GitLab服务器`/srv/gitlab/config/gitlab.rb`配置文件，修改后重启Docker容器

```
gitlab_rails['gravatar_plain_url'] = 'https://gravatar.loli.net/avatar/%{hash}?s=%{size}&d=identicon'
```
