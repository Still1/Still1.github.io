---
title: Vue Router使用
tags: [软件开发, Vue, JavaScript]
---

## 安装Vue Router插件

Vue Router3匹配Vue2

Vue Router4匹配Vue3

```shell
npm i vue-router@3
```

## Vue Router引入

在与`main.js`文件同级的目录下增加一个`pages`文件夹用于放置路由组件

在与`main.js`文件同级的目录下增加一个`router`文件夹

`router/index.js`

```js
import Vue from 'vue';
import VueRouter from 'vue-router';
import About from '../pages/About';
import Home from '../pages/Home';
import News from '../pages/News';
import Message from '../pages/Message';
import Detail from '../pages/Detail';

Vue.use(VueRouter);

const router = new VueRouter({
  routes: [
    {
      path: '/about',
      component: About,
      meta: {
        title: 'About'
      },
    }
  ]
});

export default router;
```

`main.js`

将`VueRouter`实例对象放到`Vue`实例对象的`router`属性中

```js
import Vue from 'vue'
import router from './router';
import App from './App.vue'

Vue.config.productionTip = false

new Vue({
  render: h => h(App),
  router
}).$mount('#app');
```



## 页面使用Vue Router

点击`router-link`元素控制路由跳转路径

`router-view`元素指示组件显示区域

```html
<template>
  <div>
    <div class="row">
      <div class="col-xs-2 col-xs-offset-2">
        <div class="list-group">
          <router-link class="list-group-item" active-class="active" to="/about">About</router-link>
          <router-link class="list-group-item" active-class="active" to="/home">Home</router-link>
        </div>
      </div>
      <router-view></router-view>
    </div>
  </div>
</template>
```

## 路由配置的组件与router-view之间的关联

非嵌套路由将关联App组件里的`router-view`元素

```js
const router = new VueRouter({
  routes: [
    {
      path: '/about',
      // About组件将显示到App组件的router-view元素区域
      component: About,
      meta: {
        title: 'About'
      }
    }
  ]
)};
```

嵌套路由将关联上级组件里的`router-view`元素

```js
const router =  new VueRouter({
  routes: [
    {
      path: '/home',
      component: Home,
      meta: {
        title: 'Home'
      },
      children: [
        {
          path: 'news',
          // News组件将显示到Home组件的router-view元素区域
          component: News,
          meta: {
            title: 'News'
          }
        }
      ]
    }
  ]
});
```

一个组件中定义多个`router-view`元素，需要使用`name`属性区分

```js
const router =  new VueRouter({
  routes: [
    {
      path: '/about',
      // Banner组件将显示到App组件的name属性为banner的router-view元素区域
      // Footer组件将显示到App组件的name属性为footer的router-view元素区域
      components: {
        banner: Banner,
        footer: Footer
      },
      meta: {
        title: 'About'
      }
    }
  ]
)};
```

```html
<template>
  <div>
    <div class="row">
      <div class="col-xs-2 col-xs-offset-2">
        <div class="list-group">
          <router-link class="list-group-item" active-class="active" to="/about">About</router-link>
          <router-link class="list-group-item" active-class="active" to="/home">Home</router-link>
        </div>
      </div>
      <router-view name="banner"></router-view>
      <router-view name="footer"></router-view>
    </div>
  </div>
</template>
```

## 编程式路由跳转

```js
// 后退
this.$router.back();
this.$router.go(-1)；
// 前进
this.$router.forward();
this.$router.go(1);

// 跳转
this.$router.push(path);
this.$router.replace(path);
```

## 参数传递

### query参数

#### 加入参数

```html
{% raw %}
<ul>
  <li v-for="message in messageList" :key="message.id">
    <router-link :to="`/home/message/detail?id=${message.id}&content=${message.content}`">{{message.content}}</router-link>
    <router-link :to="link(message)">{{message.content}}</router-link>
    <button @click="push(message)">push</button>
    <button @click="replace(message)">replace</button>
  </li>
</ul>
{% endraw %}
```

```js
methods: {
  push(message) {
    this.$router.push({
      path: '/home/message/detail',
      query: {
        id: message.id,
        content: message.content
      }
    });
  },
  replace(message) {
    this.$router.replace({
      path: '/home/message/detail',
      query: {
        id: message.id,
        content: message.content
      }
    });
  },
  link(message) {
    return {
      path: '/home/message/detail',
      query: {
        id: message.id,
        content: message.content
      }
    }
  }
}
```

#### 参数接收

```html
<template>
  <ul>
    <li>消息编号: {{$route.query.id}}</li>
    <li>消息内容: {{$route.query.content}}</li>
  </ul>
</template>
```

### params参数

#### 加入参数

```html
<ul>
  <li v-for="message in messageList" :key="message.id">
    <router-link :to="`/home/message/detail/${message.id}/${message.content}`">{{message.content}}</router-link>
    <router-link :to="link(message)">{{message.content}}</router-link>
    <button @click="push(message)">push</button>
    <button @click="replace(message)">replace</button>
  </li>
</ul>
```

```js
// 不能使用path属性，改成用name属性指明路由
methods: {
  push(message) {
    this.$router.push({
      name: 'detail',
      params: {
        id: message.id,
        content: message.content
      }
    });
  },
  replace(message) {
    this.$router.replace({
      name: 'detail',
      params: {
        id: message.id,
        content: message.content
      }
    });
  },
  link(message) {
    return {
      name: 'detail',
      params: {
        id: message.id,
        content: message.content
      }
    }
  }
}
```

```js
const router =  new VueRouter({
  routes: [
    {
      path: '/home',
      component: Home,
      meta: {
        title: 'Home'
      },
      children: [
        {
          path: 'message',
          component: Message,
          meta: {
            title: 'Message'
          },
          children: [
            {
              name: 'detail',
              path: 'detail/:id/:content',
              component: Detail,
              props(route) {
                return route.query;
              },
              meta: {
                title: 'Detail'
              }
            }
          ]
        }
      ]
    }
  ]
});
```

#### 参数接收

```html
<template>
  <ul>
    <li>消息编号: {{$route.params.id}}</li>
    <li>消息内容: {{$route.params.content}}</li>
  </ul>
</template>
```

### props接收参数

```js
const router = new VueRouter({
  routes: [
    {
      path: '/home',
      component: Home,
      meta: {
        title: 'Home'
      },
      children: [
        {
          path: 'message',
          component: Message,
          meta: {
            title: 'Message'
          },
          children: [
            {
              path: 'detail',
              component: Detail,
              props(route) {
                return route.query;
              },
              meta: {
                title: 'Detail'
              }
            }
          ]
        }
      ]
    }
  ]
});
```

```html
<template>
  <ul>
    <li>消息编号: {{id}}</li>
    <li>消息内容: {{content}}</li>
  </ul>
</template>

<script>
export default {
  name: 'Detail',
  props: ['id', 'content']
}
</script>
```

## 缓存路由组件

组件切换走时不销毁

```html
<keep-alive>
    <router-view></router-view>
</keep-alive>
```

指定缓存的组件

```html
<keep-alive include="Message">
	<router-view></router-view>
</keep-alive>

<keep-alive :include="['Message', 'News']">
	<router-view></router-view>
</keep-alive>
```

## 路由守卫

### VueRouter实例对象内定义

```js
const router = new VueRouter({
  routes: [
    {
      path: '/about',
      component: About,
      meta: {
        title: 'About'
      },
      beforeEnter(to, from, next) {
        if (localStorage.getItem('aboutOk') === 'ok') {
          next();
        } else {
          alert("no auth");
        }
      }
    }
  ]
});

router.beforeEach((to, from, next) => {
  if (to.meta.isAuth) {
    if (localStorage.getItem("ok") === 'ok') {
      next();
    } else {
      alert("no auth");
    }
  } else {
    next();
  }
});

router.afterEach((to) => {
  document.title = to.meta.title;
});

export default router;
```

### 组件内定义

```js
export default {
  name: 'Detail',
  beforeRouteEnter (to, from, next) {
    console.log('beforeRouteEnter');
    next();
  },
  beforeRouteLeave (to, from, next) {
    console.log('beforeRouteLeave');
    next();
  }
}
```

## 路由生命周期钩子

`activated` 路由组件切换到前台触发

`deactivated` 路由组件切换到后台触发