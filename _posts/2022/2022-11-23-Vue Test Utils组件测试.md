---
title: Vue Test Utils组件测试
tags: 
  - 实践案例
---

## 环境准备

### Jest安装

```shell
npm install --save-dev jest@26.6.3
```

安装完成后将Jest单元测试环境配置好

### vue-jest安装

```shell
npm install --save-dev vue-jest@4.0.1
```

### Vue Test Utils安装

``` shell
npm install --save-dev @vue/test-utils@1.3.3
```

## 编写组件测试

### 渲染组件

```js
mount(MainPage)
shallowMount(MainPage)
```

### mock模块

```js
jest.mock('axios')
```

### mock Vue对象

```js
const localVue = createLocalVue()  
localVue.use(ElementUI)  
shallowMount(LoginProcessPage, {  
  localVue
})
```

### mock Vue对象的属性

```js
const mockRouter = new VueRouter()
shallowMount(LoginProcessPage, {  
  mocks: {  
    $router: mockRouter  
  }  
})
```

### 等待异步

异步操作结束后再验证

```js
const mockRouter = new VueRouter()
const wrapper = await shallowMount(LoginProcessPage, {  
  mocks: {  
    $router: mockRouter  
  }  
})
expect(wrapper.vm.$router.push).toHaveBeenCalledTimes(1)
expect(wrapper.vm.$router.push.mock.calls[0][0]).toBe('/')
```

### stub异步方法

```js
const userStub = {  
  data: {  
    attributes: {  
      name: 'Test'  
    }  
  }  
}  
axios.get.mockResolvedValue(userStub)
```

### 触发事件

```js
const startButton = testObject.container.wrapper.find('#startButton')  
testObject.stub.firstLoadSentences()  
await startButton.trigger('click')
```

### 断言元素存在

```js
expect(testObject.container.wrapper.find('div.el-tree').exists()).toBeTruthy()
```

### 断言元素存在个数

```js
expect(wrapper.findAll('div.el-tree-node')).toHaveLength(2)
```

### 断言元素可见

```js
expect(wrapper.find('#startButton').isVisible()).toBeTruthy()
```

### 断言元素存在某个类

```js
expect(wrapper.find('#hintArea').classes('invisible')).toBeTruthy()
```

### 断言元素中存在文本

```js
expect(wrapper.text()).toContain('English1')
```

### 完整例子

```js
import {createLocalVue, shallowMount} from '@vue/test-utils'  
import LoginProcessPage from '@/pages/LoginProcessPage.vue'  
import axios from '@/utils/axios'  
import ElementUI from 'element-ui'  
import VueRouter from 'vue-router'  
  
jest.mock('axios')  
jest.mock('vue-router')  
  
let testObject = {  
  container: {  
  },  
  prepare: {  
    async render() {  
      const mockRouter = new VueRouter()  
      const localVue = createLocalVue()  
      localVue.use(ElementUI)  
      testObject.container.wrapper = await shallowMount(LoginProcessPage, {  
        localVue,  
        mocks: {  
          $router: mockRouter  
        }  
      })  
    },  
  
    async afterInit() {  
      testObject.stub.loadUserInformation()  
      await this.render()  
    }  
  },  
  
  stub: {  
    loadUserInformation() {  
      const userStub = {  
        data: {  
          attributes: {  
            name: 'Test'  
          }  
        }  
      }  
      axios.get.mockResolvedValue(userStub)  
    }  
  },  
  interaction: {  
  }  
}  
  
describe('Component Test: LoginProcessPage', () => {  
  afterEach(() => {  
    testObject.container = {}  
  })  
  
  test('should show information', async () => {  
    await testObject.prepare.afterInit()  
    expect(testObject.container.wrapper.find('h3').text()).toEqual('Wait a second...')  
  })  
  
  test('login succeed', async () => {  
    await testObject.prepare.afterInit()  
    expect(global.sessionStorage.getItem('nickname')).toBe('Test')  
    expect(testObject.container.wrapper.vm.$router.push).toHaveBeenCalledTimes(1)  
    expect(testObject.container.wrapper.vm.$router.push.mock.calls[0][0]).toBe('/')  
  })  
})
```

## 文档参考

* [Vue Test Utils](https://v1.test-utils.vuejs.org/zh/)
* [Jest](https://jestjs.io/docs/26.x/getting-started)