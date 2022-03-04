---
title: GitHub Pages + Jekyll + TeXt 个人博客实现
tags: 
  - GitHub Pages
  - Jekyll
  - TeXt
modify_date: 2020-05-23
---

## 前言

本文主要介绍个人博客的实现，利用GitHub Pages作为页面服务器，GitHub Pages天然支持Jekyll静态网站生成器，最后加上TeXt主题风格。该方法亦即本站的实现方法。

<!--more-->

## 前置条件

1. 拥有个人GitHub账户
2. 熟悉Git与GitHub的基本操作

## 搭建过程

### 方法一：Fork TeXt主题库

1. 使用你的GitHub账号，对TeXt主题仓库<https://github.com/kitian616/jekyll-TeXt-theme>进行`fork`操作

   ![github-fork](https://i.loli.net/2020/05/23/y3CbEPBxSsHr7Ld.jpg)

2. 更改刚`fork`过来的仓库的仓库名，以启用GitHub Pages。仓库名格式为`{username}.github.io`  `username`为你的GitHub账户登录的用户名。例如，你的GitHub用户名为`justext`，则更改仓库名为`justext.github.io`

   ![github-rename-repo](https://i.loli.net/2020/05/23/h6UuFY4m1t72Ioy.jpg)

3. 至此，基础架构已搭建完成，可以通过地址https://{username}.github.io访问博客页面

### 方法二：复制TeXt主题库

对比方法一，方法二稍微麻烦一点。但方法一由于是通过`fork`操作来创建的仓库，所以对该仓库的提交不纳入GitHub账户的贡献统计中，也就是说不影响下图的绿点统计

![github contributions](https://i.loli.net/2020/05/23/KIqdicwLN8ShbzF.png)

因此如果想把对该仓库的修改纳入贡献统计中，需要自己创建一个仓库，然后把TeXt主题库的代码放到新建仓库中，完成搭建。

1. 使用你的GitHub账户，创建一个仓库，仓库名格式为 `{username}.github.io` ，`{username}`为你的Github账户登录的用户名。例如，用户名为`sophshep`，则更改仓库名为`sophshep.github.io`

   ![create repo](https://i.loli.net/2020/05/23/nlcfrh7axC6mMTw.png)

2. 将刚新建的仓库克隆到本地

3. 下载TeXt主题仓库<https://github.com/kitian616/jekyll-TeXt-theme>的master分支代码压缩包到本地

   ![download zip](https://i.loli.net/2020/05/23/RWXhZkeYVG3DgaH.png)

4. 解压缩代码压缩包，把压缩包的内容复制到步骤2克隆的本地仓库中，并提交

5. 至此，基础架构已搭建完成，可以通过地址https://{username}.github.io访问博客页面

## 上传博客内容

我们只需要把Markdown或HTML格式的内容上传到仓库根目录下的`_posts`文件夹内即可，`_posts`文件夹内原来的示例内容可删除。上传文件的文件名需要以`yyyy-MM-dd`开头，例如本文的文件名为`2020-05-23-GitHub-Pages-plus-Jekyll-plus-TeXt.md`

上传后，稍待片刻，我们即可在博客页面看到刚上传的内容更新。

## 总结

本文介绍了通过GitHub Pages、Jekyll、TeXt快速搭建个人博客基础架构，以达到基础的博客使用效果。

## Reference

* [TeXt Theme Docs](https://tianqi.name/jekyll-TeXt-theme/docs/en/quick-start)
* [GitHub Pages](https://pages.github.com/)