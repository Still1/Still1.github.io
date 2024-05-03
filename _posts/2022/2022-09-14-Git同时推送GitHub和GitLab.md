---
title: Git同时推送GitHub和GitLab
tags: [Git]
---

## 实现方式

将GitHub和GitLab的远端地址放在同一个remote中

## 仓库准备

准备一个空的GitHub仓库和一个空的GitLab

本地先克隆GitHub仓库

## 远端仓库设置多个URL

加上GitLab远端仓库的URL

```
git remote set-url --add origin ssh://git@gitlab.oc.com:2222/oliver/languagetrainer.git
```

查看远端仓库所有URL

```
git remote get-url --all origin
```

查看结果

```
git@github.com:Still1/LanguageTrainer.git
ssh://git@gitlab.oc.com:2222/oliver/languagetrainer.git
```

