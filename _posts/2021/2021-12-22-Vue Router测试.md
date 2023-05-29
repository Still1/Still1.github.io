---
title: Vue Router测试
tags: [软件测试, JavaScript, Vue, Jest, Vue Test Utils, 组件测试]
---

## 测试代码

mock一个`VueRouter`对象，断言被调用`push`方法

```js
import {createLocalVue, mount} from '@vue/test-utils'  
import LoginPage from '@/pages/LoginPage.vue'  
import axios from '@/utils/axios'  
import VueRouter from 'vue-router'  
  
jest.mock('axios')  
jest.mock('vue-router')  
  
let testObject = {  
  container: {  
  },  
  prepare: {  
    async render() {  
      const localVue = createLocalVue()  
      localVue.use(VueRouter)  
      const mockRouter = new VueRouter()  
  
      testObject.container.wrapper = await mount(LoginPage, {  
        localVue,  
        mocks: {  
          $router: mockRouter  
        }  
      })  
    }  
  },  
  
  stub: {  
    login() {  
      const responseStub = {  
        'data': 'Test'  
      }  
      axios.post.mockResolvedValueOnce(responseStub)  
    }  
  },  
  
  interaction: {  
    async clickLoginButton() {  
      const loginButton = testObject.container.wrapper.findComponent({ref: 'loginButton'})  
      await loginButton.trigger('click')  
    }  
  }  
}  
  
describe('Component Test: LoginPage', () => {  
  afterEach(() => {  
    testObject.container = {}  
  })  
  
  test('should route to main page after clicking the login button', async () => {  
    await testObject.prepare.render()  
    testObject.stub.login()  
    await testObject.interaction.clickLoginButton()  
    expect(testObject.container.wrapper.vm.$router.push).toHaveBeenCalledWith('/')  
  })  
})
```