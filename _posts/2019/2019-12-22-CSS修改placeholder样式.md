---
title: CSS修改placeholder样式
tags: [CSS, 软件开发]
---

## 一般方法

```css
input::placeholder {
  color: red;
}

textarea::placeholder {
  color: red;
}
```

## 兼容浏览器

Chrome内核

```css
input::-webkit-input-placeholder {
  color: red;
}
```

Firefox 4~18

```css
input::-moz-placeholder {
  color: red;
}
```

Firefox 19+

```css
input::-moz-placeholder {
  color: red;
}
```

IE 10+

```css
input::-ms-input-placeholder {
  color: red;
}
```