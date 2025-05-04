---
title: 解决EasyUI的tree组件中，原来是叶子节点的节点，加了下级节点后，无法reload的问题
tags: [EasyUI, 问题解决]
---

## 问题描述

EasyUI的tree组件中，原来是叶子节点的节点，加了下级节点后，无法reload

## 问题原因

tree组件`reload`某一个节点，只能支持非叶子节点，对叶子节点无效

## 解决方法

`reload`它的父节点，但会产生新的问题，原来选中的节点由于`reload`过，选中状态没有了，而且`reload`过的节点与`reload`前的节点内部会有不同，用变量记录下来的选中节点的`node.target`已经无法进行`reload`操作或`select`操作

解决新问题方法

在`onLoadSuccess`方法中，利用变量记录下来的选中的节点`node.id`，寻找`reload`后相对应的节点

```javascript
//找出与reload前选中的树节点相对应的reload后的树节点
var selectedNode = $('#categoryTree').tree('find', currentSelectedNode.id);
$('#categoryTree').tree('select', selectedNode.target);
```