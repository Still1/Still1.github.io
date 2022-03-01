---
title: JQuery的each方法实现循环的break和continue
tags: 
  - JQuery
modify_date: 2018-03-31
---

## 实现方法

break：使用`return false;`

continue：使用`return true;`

<!--more-->

## 代码例子

```javascript
getConditionUrl: function() {
    var conditionArray = new Array();
    $('#conditionDiv div>input').each(function(index, element) {
        var conditionObject = new Object();
        var elementObject = $(element);
        if (elementObject.hasClass('easyui-textbox')) {
            conditionObject.value = elementObject.textbox('getValue');
            if (conditionObject.value == undefined || conditionObject.value) {
                return true;
            }
            conditionObject.name = elementObject.attr('textboxname');
        }
        conditionObject.op = elementObject.attr('data-operation');
        conditionArray.push(conditionObject);
    });
    return JSON.stringify(conditionArray);
}
```

