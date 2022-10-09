---
title: Jekyll TeXt 个人博客样式配置
tags: 
  - 实践案例
---

## 修改_config.yml

<!--more-->

配置TeXt提供的主题样式

```yaml
text_skin: chocolate
```

配置TeXt提供的代码高亮配色

```yaml
highlight_theme: tomorrow-night-eighties
```

配置博客标题

```yaml
title: Oliver's Blog
```

配置单页文章数

```yaml
paginate: 10
```



## 配置favicon

使用https://realfavicongenerator.net/ 生成favicon，并将生成的文件放到项目根目录中

修改`/_includes/head/favicon.html`文件，注释原来的所有代码，并加入上一步生成的代码

```html
<link rel="apple-touch-icon" sizes="120x120" href="/apple-touch-icon.png">
<link rel="icon" type="image/png" sizes="32x32" href="/favicon-32x32.png">
<link rel="icon" type="image/png" sizes="16x16" href="/favicon-16x16.png">
<link rel="manifest" href="/site.webmanifest">
<link rel="mask-icon" href="/safari-pinned-tab.svg" color="#5bbad5">
<meta name="msapplication-TileColor" content="#da532c">
<meta name="theme-color" content="#ffffff">
```

https://www.aconvert.com/cn/image/png-to-svg/ 转换favicon为SVG格式

使用转换的SVG文件替换原来的`/_includes/svg/logo.svg`文件



## 文章显示相关设置

可在`_config.yml`中的defaults下设置全局属性，更新`_config.yml`需要重启服务器。

也可在文章的front matter中设置，这样设置只针对本篇文章。

### 建议全局配置项

```yaml
show_edit_on_github: false
show_subscribe: false
license: false
```

### 建议文章front matter配置项

```yaml
modify_date: 2020-05-18
tags: 
  - Git
  - GitHub
```



## 字体相关配置

### 修改字体

修改`\_sass\common\_variables.scss`文件

`font-family`修改页面字体，`font-family-code`修改代码块字体

```scss
  font-family:            (-apple-system, BlinkMacSystemFont, "Microsoft YaHei", "Segoe UI", "Helvetica Neue", Helvetica, Verdana, Arial, sans-serif),
  font-family-code:       ("PT Mono", Monaco, Menlo, "Source Code Pro", Consolas, "Courier New", "lucida console", "Microsoft YaHei", monospace),
```

### 修改代码块字体大小和行高

修改`_sass\common\_reset.scss`文件

```scss
code {
    font-size: map-get($base, font-size-sm);
    line-height: map-get($base, line-height-lg);
}
```



## 解决kramdown解析markdown把英文上撇号、单引号转换为中文单引号的问题

修改`/_config.yml`文件，在文件末尾加上

```
kramdown:
  smart_quotes: ["apos", "apos", "quot", "quot"]
```



## 解决CDN访问过慢的问题

### 浏览器报错

> [Intervention] Slow network is detected. See https://www.chromestatus.com/feature/5636954674692096 for more details. Fallback font will be used while loading: https://use.fontawesome.com/releases/v5.0.13/webfonts/fa-solid-900.woff2

### 解决方法

修改`/_data/variables.yml`文件，将正在使用的速度过慢的Font Awesome CDN配置注释掉，默认配置使用sources/bootcdn下的配置项

```yaml
sources:
  bootcdn:
    # font_awesome: 'https://use.fontawesome.com/releases/v5.0.13/css/all.css'
```

下载fontawesome，将css、webfonts文件夹复制到项目代码中，路径自己选择

如放到`/assets/css/fontawesome`中然后在`/_includes/head/custom.html`中加入

```html
<link href="/assets/css/fontawesome/css/all.css" rel="stylesheet">
```

