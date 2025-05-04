---
title: 解决EasyUI datagrid行编辑，非手动填写改动的行，无法在getChanges updated数组中取得的问题
tags: [EasyUI, 问题解决]
---

## 问题描述

```javascript
for(var property in attributeRow) {
    var editor = $("#attributeDatagrid").datagrid("getEditor", {index: currentEditRowIndex, field: property});
    var editorTarget = editor.target;
    editorTarget.textbox('setValue', attributeRow[property]);
}
```

利用此方法将变动的值设置到对应的editor中，页面上看表格的值都已改变，`getRows`中的数据也已改变，但在`getChanges`时无法获取行

尝试过`editorTarget.val(attributeRow[property]);` 此方法不会改变页面输入框里的值，`getRows`中的数据也不变，但在`getChanges`时会有对应的行出现，但数据是未更新的。
尝试过`editor.actions.setValue(editorTarget, attributeRow[property]`，效果与`editorTarget.textbox('setValue', attributeRow[property])`基本相同，未看出区别

## 解决方法

表格中随便加上一个隐藏列，此字段是查询语句没有查出来的，这会导致曾经进入过编辑状态且完成编辑的行都会在`getChanges`中`updated`数组找到对应的行。缺点是有可能导致额外的数据提交，且列表多了一列多余的隐藏列，取巧方法，并不完美