---
title: JavaScript实现面向对象
tags: [JavaScript]
---

## 构造方法加原型对象实现

构造方法负责初始化变量

原型对象负责定义对象共享的方法和有默认值的属性

```javascript
function Animal(name) {
    this.name = name
}

Animal.prototype = {
    name: '',
    printName: function() {
        console.log(this.name)
    }
}
```



## 继承

通过原型对象来实现

```javascript
function Dog(name) {
    this.name = name
}

Dog.prototype = new Animal('')
Dog.prototype.bark = function() {
    console.log('bark')
}
```

