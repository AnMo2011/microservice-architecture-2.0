### 概述

Vue Router 是 Vue.js 官方的路由管理器。它和 Vue.js 的核心深度集成，让构建单页面应用变得易如反掌。包含的功能有：

- 嵌套的路由/视图表

- 模块化的、基于组件的路由配置

- 路由参数、查询、通配符

- 基于 Vue.js 过渡系统的视图过渡效果

- 细粒度的导航控制

- 带有自动激活的 CSS class 的链接

- HTML5 历史模式或 hash 模式，在 IE9 中自动降级

- 自定义的滚动条行为

### 安装

vue-router 是一个插件包，所以我们还是需要用 npm/cnpm 来进行安装的。打开命令行工具，进入你的项目目录，输入下面命令。

```
npm install vue-router --save-dev --registry=https://registry.npm.taobao.org

# 输出如下
npm WARN ajv-keywords@3.4.1 requires a peer of ajv@^6.9.1 but none is installed. You must install peer dependencies yourself.
npm WARN optional SKIPPING OPTIONAL DEPENDENCY: fsevents@1.2.9 (node_modules\fsevents):
npm WARN notsup SKIPPING OPTIONAL DEPENDENCY: Unsupported platform for fsevents@1.2.9: wanted {"os":"darwin","arch":"any"} (current: {"os":"win32","arch":"x64"})

+ vue-router@3.0.7
added 1 package from 1 contributor and audited 11766 packages in 11.33s
found 10 vulnerabilities (6 moderate, 4 high)
  run `npm audit fix` to fix them, or `npm audit` for details
```

如果在一个模块化工程中使用它，必须要通过 **Vue.use()** 明确地安装路由功能：

```
import Vue from 'vue'
import VueRouter from 'vue-router'

Vue.use(VueRouter);
```

### 使用

以下案例在 vue-cli 项目中使用 vue-router

### 创建组件页面

创建一个名为 **src/components** 的目录专门放置我们开发的 Vue 组件，在 **src/components** 目录下创建一个名为 **Content.vue** 的组件，代码如下：

```
<template>
    <div>
      我是内容页
    </div>
</template>

<script>
    export default {
        name: "Content"
    }
</script>

<style>
  #app {
    font-family: 'Avenir', Helvetica, Arial, sans-serif;
    -webkit-font-smoothing: antialiased;
    -moz-osx-font-smoothing: grayscale;
    text-align: center;
    color: #2c3e50;
    margin-top: 60px;
  }
</style>
```

### 安装路由

创建一个名为 **src/router** 的目录专门放置我们的路由配置代码，在 **src/router** 目录下创建一个名为 **index.js** 路由配置文件，代码如下：

```
import Vue from 'vue'

// 导入路由插件
import Router from 'vue-router'

// 导入上面定义的组件
import Content from '@/components/Content'

// 安装路由
Vue.use(Router);

// 配置路由
export default new Router({
  routes: [
    {
      // 路由路径
      path: '/content',
      // 路由名称
      name: 'Content',
      // 跳转到组件
      component: Content
    }
  ]
});
```

## 配置路由

修改 **main.js** 入口文件，增加配置路由的相关代码

```
import Vue from 'vue'
import App from './App'
// 导入上面创建的路由配置目录
import router from './router'

Vue.config.productionTip = false;

new Vue({
  el: '#app',
  // 配置路由
  router,
  components: { App },
  template: '<App/>'
});
```

### 使用路由

修改 **App.vue** 组件，代码如下：

```
<template>
  <div id="app">
    <router-link to="/">首页</router-link>
    <router-link to="/content">内容</router-link>
    <router-view></router-view>
  </div>
</template>

<script>
export default {
  name: 'App'
}
</script>

<style>
  #app {
    font-family: 'Avenir', Helvetica, Arial, sans-serif;
    -webkit-font-smoothing: antialiased;
    -moz-osx-font-smoothing: grayscale;
    text-align: center;
    color: #2c3e50;
    margin-top: 60px;
  }
</style>
```

说明：

- **router-link**： 默认会被渲染成一个 <a> 标签，to 属性为指定链接

- **router-view**： 用于渲染路由匹配到的组件

### 效果演示

![](../img/10-00000036.gif)
