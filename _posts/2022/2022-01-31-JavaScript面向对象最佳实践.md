---
title: JavaScript面向对象最佳实践
tags: 
  - 知识总结
---

## 构造方法加原型对象实现

构造方法负责对象各自独立的属性

原型对象负责对象共享的属性和方法

<!--more-->

```javascript
function Animal(name) {
    this.name = name;
}


Animal.prototype = {
    commonAttr: 'common',
    printCommonAttr: function() {
        console.log(this.commonAttr);
    },
    printName: function() {
        console.log(this.name);
    }
};
```

## 继承

通过原型对象的类型来实现

```javascript
function Dog(name, type) {
    this.name = name;
    this.type = type;
}

Dog.prototype = new Animal('Rocky');
Dog.prototype.printType = function() {
    console.log(this.type);
}
```

