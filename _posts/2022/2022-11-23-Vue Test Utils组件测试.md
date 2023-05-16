---
title: Vue Test Utils组件测试
tags: [软件测试, 组件测试, JavaScript, Vue, Vue Test Utils, Jest]
---

## 环境准备

### Jest安装

```shell
npm install --save-dev jest@26.6.3
```

安装完成后将Jest单元测试环境配置好

> [Jest单元测试](https://blog.oliverclio.com/2022/10/23/Jest%E5%8D%95%E5%85%83%E6%B5%8B%E8%AF%95.html)

### vue-jest安装

```shell
npm install --save-dev vue-jest@4.0.1
```

### Vue Test Utils安装

``` shell
npm install --save-dev @vue/test-utils@1.3.3
```

## 编写组件测试

### 准备

#### 渲染组件

```js
mount(MainPage)
shallowMount(MainPage)
```

#### 访问storage

```js
global.sessionStorage.setItem('userId', '1')
```

#### mock模块

```js
jest.mock('axios')
```

#### localVue对象

```js
const localVue = createLocalVue()  
localVue.use(ElementUI)  
shallowMount(LoginProcessPage, {  
  localVue
})
```

#### mock Vue全局属性

```js
const mockRouter = new VueRouter()
shallowMount(LoginProcessPage, {  
  mocks: {  
    $router: mockRouter  
  }  
})
```

#### stub组件

```js
mount(StartComponent, {  
  stubs: ['font-awesome-icon']  
}
```

#### stub props属性

```js
mount(StartComponent, {  
  propsData: {  
    corpusName: 'English'  
  }
}
```

#### 等待异步渲染

```js
const mockRouter = new VueRouter()
const wrapper = await shallowMount(LoginProcessPage, {  
  mocks: {  
    $router: mockRouter  
  }  
})
```

#### stub异步方法

```js
jest.mock('@/utils/axios')

const userStub = {  
  data: {  
    attributes: {  
      name: 'Test'  
    }  
  }  
}  
axios.get.mockResolvedValue(userStub)
```

#### 触发事件

```js
await wrapper.find('#startButton').trigger('click')
await wrapper.find('input.el-input__inner').trigger('keyup.enter')
```

#### 抛出事件

```js
wrapper.findComponent({ref: 'loginButton'}).vm.$emit('click')
await wrapper.vm.$nextTick()
```

#### 模拟输入

```js
// 只支持原生input
await wrapper.find('input.el-input__inner').setValue('text')
```

### 断言

#### 断言元素存在

```js
expect(wrapper.find('div.el-tree').exists()).toBeTruthy()
```

#### 断言组件存在

```js
expect(wrapper.findComponent({ref: 'startButton'}).exists()).toBeTruthy()
```

#### 断言元素存在个数

```js
expect(wrapper.findAll('div.el-tree-node')).toHaveLength(2)
```

#### 断言元素可见

```js
expect(wrapper.find('#startButton').isVisible()).toBeTruthy()
```

#### 断言元素存在某个类

```js
expect(wrapper.find('#hintArea').classes('invisible')).toBeTruthy()
```

#### 断言元素中存在文本

```js
expect(wrapper.text()).toContain('English1')
expect(wrapper.text()).toBe('English1')
```

#### 断言方法被调用过

```js
expect(wrapper.vm.$router.push).toHaveBeenCalledWith('/')
expect(wrapper.vm.$router.push).toHaveBeenCalledTimes(1)  
expect(wrapper.vm.$router.push.mock.calls[0][0]).toBe('/')
```

### 测试例子

#### 代码框架

```js
const testObject = {  
  // 用于存放Wrapper或WrapperArray
  container: {  
  },  
  // 用于进行测试前的准备操作，调用stub方法或其它prepare方法，预先存放Wrapper或WrapperArray到container对象
  prepare: {  
  },  
  // 用于进行stub相关操作
  stub: {  
  }  
}

describe('Component Test', () => {  
  afterEach(() => {  
    testObject.container = {}  
  })  

  // 测试用例调用prepare方法准备环境，然后使用container中的对象进行断言即可
  test('test', async () => {
  })  
})
```

#### 测试文本、元素或组件

```js
import {createLocalVue, mount} from '@vue/test-utils'  
import LoginPage from '@/pages/LoginPage.vue'  
import ElementUI from 'element-ui'  
  
const testObject = {  
  container: {  
  },  
  prepare: {  
    async render() {  
      const localVue = createLocalVue()  
      localVue.use(ElementUI)  
  
      testObject.container.wrapper = await mount(LoginPage, {  
        localVue,  
        stubs: ['font-awesome-icon']  
      })  
  
      testObject.container.header = testObject.container.wrapper.find('h3')  
      testObject.container.gitHubButton = testObject.container.wrapper.findComponent({ref: 'gitHubButton'})  
      testObject.container.username = testObject.container.wrapper.findComponent({ref: 'username'})  
      testObject.container.password = testObject.container.wrapper.findComponent({ref: 'password'})  
      testObject.container.loginButton = testObject.container.wrapper.findComponent({ref: 'loginButton'})  
    }
  },  
  
  stub: {  
  }  
}  
  
describe('Component Test: LoginPage', () => {  
  afterEach(() => {  
    testObject.container = {}  
  })  
  
  test('should show header', async () => {  
    await testObject.prepare.render()  
    expect(testObject.container.header.text()).toBe('Login with GitHub')  
  })  
  
  test('should show GitHub login button', async () => {  
    await testObject.prepare.render()  
    expect(testObject.container.gitHubButton.exists()).toBeTruthy()  
    expect(testObject.container.gitHubButton.isVisible()).toBeTruthy()  
  })  
  
  test('should show a login input form', async () => {  
    await testObject.prepare.render()  
    expect(testObject.container.username.exists()).toBeTruthy()  
    expect(testObject.container.username.isVisible()).toBeTruthy()  
    expect(testObject.container.password.exists()).toBeTruthy()  
    expect(testObject.container.password.isVisible()).toBeTruthy()  
    expect(testObject.container.loginButton.exists()).toBeTruthy()  
    expect(testObject.container.loginButton.isVisible()).toBeTruthy()  
  })  
})
```

#### stub外部数据

```js
import {createLocalVue, mount} from '@vue/test-utils'  
import LoginPage from '@/pages/LoginPage.vue'  
import ElementUI from 'element-ui'  
import axios from '@/utils/axios'  
  
jest.mock('@/utils/axios')  
  
const testObject = {  
  container: {  
  },  
  prepare: {  
    async render() {  
      const localVue = createLocalVue()  
      localVue.use(ElementUI)  
  
      testObject.container.wrapper = await mount(LoginPage, {  
        localVue,
        stubs: ['font-awesome-icon']  
      })  
  
      testObject.container.loginButton = testObject.container.wrapper.findComponent({ref: 'loginButton'})  
    },  
  
    async afterLoginButtonClickEvent() {  
      await this.render()  
      testObject.stub.login()  
      testObject.container.loginButton.vm.$emit('click')  
      await testObject.container.wrapper.vm.$nextTick()
    }  
  },  
  
  stub: {  
    login() {  
      const responseStub = {  
        data: {  
          data: {  
            nickname: 'Test',  
            loginDays: 3,  
            straightLoginDays: 1  
          }  
        }  
      }  
      axios.post.mockResolvedValueOnce(responseStub)  
    }  
  }  
}  
  
describe('Component Test: LoginPage', () => {  
  afterEach(() => {  
    testObject.container = {}  
  })  
  
  test('should get nickname after login button click event', async () => {  
    await testObject.prepare.afterLoginButtonClickEvent()  
    expect(global.sessionStorage.getItem('nickname')).toBe('Test')  
  })  
})
```

#### 测试Vue Router

```js
import {createLocalVue, mount} from '@vue/test-utils'  
import LoginPage from '@/pages/LoginPage.vue'  
import ElementUI from 'element-ui'  
import VueRouter from 'vue-router'  
  
jest.mock('vue-router')  
  
const testObject = {  
  container: {  
  },  
  prepare: {  
    async render() {  
      const localVue = createLocalVue()  
      localVue.use(ElementUI)  
      const mockRouter = new VueRouter()  
  
      testObject.container.wrapper = await mount(LoginPage, {  
        localVue,  
        mocks: {  
          $router: mockRouter  
        },  
        stubs: ['font-awesome-icon']  
      })  
  
      testObject.container.loginButton = testObject.container.wrapper.findComponent({ref: 'loginButton'})  
    },  
  
    async afterLoginButtonClickEvent() {  
      await this.render()  
      testObject.stub.login()  
      testObject.container.loginButton.vm.$emit('click')  
      await testObject.container.wrapper.vm.$nextTick()
    }  
  },  
  
  stub: {  
    login() {  
      const responseStub = {  
        data: {  
          data: {  
            nickname: 'Test',  
            loginDays: 3,  
            straightLoginDays: 1  
          }  
        }  
      }  
      axios.post.mockResolvedValueOnce(responseStub)  
    }  
  }  
}  
  
describe('Component Test: LoginPage', () => {  
  afterEach(() => {  
    testObject.container = {}  
  })  
  
  test('should route to main page after login button click event', async () => {  
    await testObject.prepare.afterLoginButtonClickEvent()  
    expect(testObject.container.wrapper.vm.$router.push).toHaveBeenCalledWith('/')  
  })  
})
```

#### 测试事件抛出

```js
import {createLocalVue, mount} from '@vue/test-utils'  
import StartComponent from '@/components/StartComponent.vue'  
import ElementUI from 'element-ui'  
  
const testObject = {  
  container: {  
  },  
  prepare: {  
    async render() {  
      const localVue = createLocalVue()  
      localVue.use(ElementUI)  
  
      testObject.container.wrapper = await mount(StartComponent, {  
        localVue,  
        propsData: {  
          corpusName: 'English'  
        },  
        stubs: ['font-awesome-icon']  
      })  
      testObject.container.startButton = testObject.container.wrapper.findComponent({ref: 'startButton'})  
    },  
  
    async afterStartButtonClickEvent() {  
      await this.render()  
      testObject.container.startButton.vm.$emit('click')  
      await testObject.container.wrapper.vm.$nextTick()
    }  
  }  
}  
  
describe('Component Test: StartComponent', () => {  
  afterEach(() => {  
    testObject.container = {}  
  })  
  
  test('should emit startButtonClick event after clicking the start button', async () => {  
    await testObject.prepare.afterStartButtonClickEvent()  
    expect(testObject.container.wrapper.emitted('startButtonClick')).toBeTruthy()  
    expect(testObject.container.wrapper.emitted('startButtonClick').length).toBe(1)  
  })  
})
```

#### 原生事件触发

```js
import {createLocalVue, mount} from '@vue/test-utils'  
import ElementUI from 'element-ui'  
import ExerciseComponent from '@/components/ExerciseComponent'  
import Vue from 'vue'  
  
const testObject = {  
  container: {  
  },  
  prepare: {  
    async render() {  
      const localVue = createLocalVue()  
      localVue.use(ElementUI)  
  
      testObject.container.wrapper = await mount(ExerciseComponent, {  
        localVue,  
        propsData: {  
          corpusName: 'English',  
          sentenceArray: [  
            {  
              id: 1,  
              corpusId: 3,  
              text: 'test',  
              hint: '测试',  
              sort: 1  
            },  
            {  
              id: 2,  
              corpusId: 3,  
              text: 'test2',  
              hint: '测试2',  
              sort: 2  
            }  
          ]  
        }
      })  
  
      testObject.container.sentenceInput = testObject.container.wrapper.find('input.el-input__inner')  
      testObject.container.hintArea = testObject.container.wrapper.find('#hintArea')  
    }
    async afterEnterWrongAnswer() {  
      await this.render()  
      await testObject.container.sentenceInput.setValue('t')  
      // 无法使用组件的$emit模拟
      await testObject.container.sentenceInput.trigger('keyup.enter')  
    }
  }  
}  
  
describe('Component Test: ExerciseComponent', () => {  
  afterEach(() => {  
    testObject.container = {}  
  })  
  
  test('the color of the input field should turn red and should show hint when inputting a wrong answer', async () => {  
    await testObject.prepare.afterEnterWrongAnswer()  
    expect(testObject.container.sentenceInput.classes('wrongInput')).toBeTruthy()  
    expect(testObject.container.hintArea.classes('invisible')).toBeFalsy()  
  })
})
```

#### 复合键盘事件触发

```js
await testObject.container.sentenceInput.trigger('keyup', {  
  ctrlKey: true,  
  key: 'ArrowRight',  
  keyCode: 39  
})
```

#### 测试事件总线接收

```js
import {createLocalVue, mount} from '@vue/test-utils'  
import ElementUI from 'element-ui'  
import ExerciseComponent from '@/components/ExerciseComponent'  
import Vue from 'vue'  
  
const testObject = {  
  container: {  
  },  
  prepare: {  
    async render() {  
      const localVue = createLocalVue()  
      localVue.use(ElementUI)  
  
      testObject.container.wrapper = await mount(ExerciseComponent, {  
        localVue,  
        propsData: {  
          corpusName: 'English',  
          sentenceArray: [  
            {  
              id: 1,  
              corpusId: 3,  
              text: 'test',  
              hint: '测试',  
              sort: 1  
            },  
            {  
              id: 2,  
              corpusId: 3,  
              text: 'test2',  
              hint: '测试2',  
              sort: 2  
            }  
          ]  
        },  
        mocks: {  
          $bus: new Vue()  
        }  
      })  
  
      testObject.container.sentenceInput = testObject.container.wrapper.find('input.el-input__inner')  
      testObject.container.hintArea = testObject.container.wrapper.find('#hintArea')  
    },  
    async afterEnterRightAnswer() {  
      await this.render()  
      await testObject.container.sentenceInput.setValue('test')  
      await testObject.container.sentenceInput.trigger('keyup.enter')  
    },  
    async afterResetSentenceStatusEvent() {  
      await testObject.prepare.afterEnterRightAnswer()  
      testObject.container.wrapper.vm['$bus'].$emit('resetSentenceStatus')  
      // 或者直接调用事件处理方法
      // testObject.container.wrapper.vm['handleResetSentenceStatus']()
      await testObject.container.wrapper.vm.$nextTick()  
    }  
  }  
}  
  
describe('Component Test: ExerciseComponent', () => {  
  afterEach(() => {  
    testObject.container = {}  
  })  
  
  test('should reset the sentence information after resetSentenceStatus event emitted', async () => {  
    await testObject.prepare.afterResetSentenceStatusEvent()  
    expect(testObject.container.sentenceInput.classes('wrongInput')).toBeFalsy()  
    expect(testObject.container.sentenceInput.classes('rightInput')).toBeFalsy()  
    expect(testObject.container.hintArea.classes('invisible')).toBeTruthy()  
    expect(testObject.container.wrapper.vm['sentenceInput']).toBe('')  
    expect(testObject.container.wrapper.vm['greenLight']).toBeFalsy()  
  })  
})
```

#### 测试事件总线抛出

`bus.js`
```js
import Vue from 'vue'  
  
function MockBus() {}  
MockBus.prototype = new Vue()  

export default MockBus
```

```js
import {createLocalVue, mount} from '@vue/test-utils'  
import MainPage from '@/pages/MainPage.vue'  
import axios from '@/utils/axios'  
import ElementUI from 'element-ui'  
import StartComponent from '@/components/StartComponent'  
import ExerciseComponent from '@/components/ExerciseComponent'  
import MockBus from '../mock/bus'  
  
jest.mock('axios')  
jest.mock('../mock/bus')  
  
let testObject = {  
  container: {  
  },  
  prepare: {  
    async render() {  
      testObject.stub.firstLoadCorpusTreeNode()  
      global.sessionStorage.setItem('nickname', 'Test Nickname')  
      global.sessionStorage.setItem('loginDays', '3')  
      global.sessionStorage.setItem('straightLoginDays', '1')  
  
      const localVue = createLocalVue()  
      localVue.use(ElementUI)  
  
      testObject.container.wrapper = await mount(MainPage, {  
        localVue,  
        mocks: {  
          $bus: new MockBus()  
        },  
        stubs: ['font-awesome-icon']  
      })  
  
      testObject.container.dialog = testObject.container.wrapper.findComponent({ref: 'userLoginStatDialog'})  
      testObject.container.startComponent = testObject.container.wrapper.findComponent(StartComponent)  
      testObject.container.tree = testObject.container.wrapper.find('div.el-tree')  
      testObject.container.treeItemArray = testObject.container.wrapper.findAll('div.el-tree-node')  
      testObject.container.firstTreeItem = testObject.container.treeItemArray.at(0)  
      testObject.container.firstLabel = testObject.container.firstTreeItem.find('span.el-tree-node__label')  
      testObject.container.secondTreeItem = testObject.container.treeItemArray.at(1)  
      testObject.container.secondLabel = testObject.container.secondTreeItem.find('span.el-tree-node__label')  
    },  
  
    async afterClickingNodeOfCorpusTree() {  
      await this.render()  
      testObject.stub.secondLoadCorpusTreeNode()  
      await testObject.container.firstTreeItem.trigger('click')  
      testObject.container.treeNodeChildren = testObject.container.firstTreeItem.find('div.el-tree-node__children')  
      testObject.container.labelArray = testObject.container.treeNodeChildren.findAll('.el-tree-node__label')  
    },  
  
    async afterClickingLeafNodeOfCorpusTree() {  
      await this.afterClickingNodeOfCorpusTree()  
      testObject.stub.loadCorpusTreeLeafNode()  
      await testObject.container.labelArray.at(0).trigger('click')  
    },  
  
    async afterStartButtonClickEvent() {  
      await this.afterClickingLeafNodeOfCorpusTree()  
      testObject.stub.firstLoadSentences()  
      testObject.container.startComponent.vm.$emit('startButtonClick')  
      testObject.container.exerciseComponent = testObject.container.wrapper.findComponent(ExerciseComponent)  
    }
  },  
  
  stub: {  
    firstLoadCorpusTreeNode() {  
      const corpusStub = {  
        data: {  
          data: [  
            {  
              id: 1,  
              name: 'English',  
              parentId: 0,  
              sort: 1  
            },  
            {  
              id: 2,  
              name: 'French',  
              parentId: 0,  
              sort: 2  
            }  
          ]  
        }  
      }  
      axios.get.mockResolvedValueOnce(corpusStub)  
    },  
  
    secondLoadCorpusTreeNode() {  
      const corpusStub = {  
        data: {  
          data: [  
            {  
              id: 3,  
              name: 'English1',  
              parentId: 1,  
              sort: 1  
            },  
            {  
              id: 4,  
              name: 'English2',  
              parentId: 1,  
              sort: 2  
            }  
          ]  
        }  
      }  
      axios.get.mockResolvedValueOnce(corpusStub)  
    },  
  
    loadCorpusTreeLeafNode() {  
      const corpusStub = {data: {data: []}}  
      axios.get.mockResolvedValueOnce(corpusStub)  
    },  
  
    firstLoadSentences() {  
      const sentencesStub = {  
        data: {  
          data: [  
            {  
              id: 1,  
              corpusId: 3,  
              text: 'text',  
              hint: '测试',  
              sort: 1  
            },  
            {  
              id: 2,  
              corpusId: 3,  
              text: 'text2',  
              hint: '测试2',  
              sort: 2  
            }  
          ]  
        }  
      }  
      axios.get.mockResolvedValueOnce(sentencesStub)  
    }  
  }  
}  
  
describe('Component Test: MainPage', () => {  
  afterEach(() => {  
    testObject.container = {}  
  })  
    
  test('should emit resetSentenceStatus event to bus after startButtonClick event emitted', async () => {  
    await testObject.prepare.afterStartButtonClickEvent()  
    expect(testObject.container.wrapper.vm['$bus'].$emit).toHaveBeenCalledWith('resetSentenceStatus')  
  })  
})
```

## 文档参考

* [Vue Test Utils](https://v1.test-utils.vuejs.org/zh/)
* [Jest](https://jestjs.io/docs/26.x/getting-started)