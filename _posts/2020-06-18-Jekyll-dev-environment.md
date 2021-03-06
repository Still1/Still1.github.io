---
title: Jekyll本地开发环境搭建
tags: 
  - Jekyll
modify_date: 2020-06-19
---

## 前言

在以下的情景中，我们可能需要在本地搭建Jekyll开发环境：

* 在博客发布之前，希望在本地预览发布效果
* 对主题进行修改，在本地进行调试
* 生成Jekyll的默认站点代码
* 使用需要运行`gem`命令生成站点代码的主题
* 开发自己的Jekyll主题

接下来本文将介绍如何在本地搭建Jekyll开发环境

<!--more-->

## 前置条件

1. 对Jekyll有基本的了解

## 搭建过程

### 安装

1. 下载并安装Ruby Installer

   [Ruby Installer下载地址](https://rubyinstaller.org/downloads/)

   {:.success}

   :heavy_check_mark:选择WITH DEVKIT类别下的官方建议下载版本

   ![Ruby Installer](https://oliver-blog.oss-cn-shenzhen.aliyuncs.com/20200619101607.png)

   {:.warning}

   :warning:选择最新版本可能会带来更大的不兼容风险。是否出现兼容性问题与开发主题时使用的构件版本相关。

2. 安装完成后弹出如下图的提示，直接按回车。若不小心关掉了，可以运行`ridk install`命令

   ![MSYS2 Install](https://oliver-blog.oss-cn-shenzhen.aliyuncs.com/20200619101924.png)

   安装完成后，再次按回车即可

3. 安装Jekyll

   打开cmd命令行，运行`gem install jekyll bundler`

   ![Jekyll Install](https://oliver-blog.oss-cn-shenzhen.aliyuncs.com/20200619102329.png)

   安装完成后，运行`jekyll --version`，如果显示出当前安装的Jekyll版本，则证明安装成功

   ![Jekell version](https://oliver-blog.oss-cn-shenzhen.aliyuncs.com/20200619102501.png)

### 生成Jekyll站点代码

如果没有任何的Jekyll站点代码，可以先生成站点代码。如果已有站点代码，则可以跳过此步骤

#### 生成Jekyll默认站点代码

1. 打开cmd命令行，在想要存放站点代码的目录下，运行`jekyll new .`，若该目录非空，则运行`jekyll new . --force`，强行覆盖原有内容。

   ![Jekyll new](https://oliver-blog.oss-cn-shenzhen.aliyuncs.com/20200619103557.png)

2. 完成后，当前目录下已经生成好默认的站点代码

#### 生成其它主题的站点代码

根据主题的文档指示来生成。

Jekyll主题发布网站：

* <https://jekyllrb.com/docs/themes/>
* <https://jekyllthemes.io/>
* <http://jekyllthemes.org/>
* <https://jamstackthemes.dev/ssg/jekyll/>
* <https://jekyll-themes.com/>

### 启动

1. 打开cmd命令行，在Jekyll站点代码的根目录下，运行`bundle exec jekyll serve`
2. 待启动完成后，即可通过<http://localhost:4000/>访问博客

## 总结

至此，Jekyll本地开发的环境搭建完毕。

## Reference

* [Jekyll on Windows](https://jekyllrb.com/docs/installation/windows/)