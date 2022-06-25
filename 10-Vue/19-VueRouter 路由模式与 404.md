### 路由模式

路由模式有两种

- hash：路径带 # 符号，如 http://localhost/#/login

- history：路径不带 # 符号，如 http://localhost/login

修改路由配置，代码如下：

```
export default new Router({
  mode: 'history',
  routes: [
  ]
});
```

### 处理 404

创建一个名为 **NotFound.vue** 的视图组件，代码如下：

```
<template>
    <div>
      页面不存在，请重试！
    </div>
</template>

<script>
    export default {
        name: "NotFount"
    }
</script>

<style scoped>
</style>
```

修改路由配置，代码如下：

```
    {
      path: '*',
      component: NotFound
    }
```
