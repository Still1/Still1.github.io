---
title: JavaScript调用eval方法将JSON字符串解析为对象
tags: 
  - 实践案例
---

```javascript
eval("({ id: 1, name: 'n_1' })");
```

必须要在字符串的两端加上小括号，否则`eval`会把大括号解析为代码块的符号，导致解析错误