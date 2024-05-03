---
title: Vue Mixins使用
tags: [Vue]
---

## 被引用的Mixins

`src/mixin/modeText.js`

```js
export default {  
  computed: {  
    modeText() {  
      if (this.mode === 'learning') {  
        return '学习模式'  
      } else if (this.mode === 'exercise') {  
        return '练习模式'  
      } else if (this.mode === 'test') {  
        return '测试模式'  
      } else {  
        return ''  
      }  
    }  
  }  
}
```

## 引入Mixins

```js
import modeText from '@/mixin/modeText'

export default {  
  name: 'ExerciseComponent',  
  mixins: [modeText]
}
```