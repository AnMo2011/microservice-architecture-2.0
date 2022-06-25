### 参数传递

我们经常需要把某种模式匹配到的所有路由，全都映射到同个组件。例如，我们有一个 User 组件，对于所有 ID 各不相同的用户，都要使用这个组件来渲染。此时我们就需要传递参数了；

### 使用路径匹配的方式

- 修改路由配置，主要是在 **path** 属性中增加了 **:id** 这样的占位符

```
{path: '/user/profile/:id', name:'UserProfile', component: UserProfile}
```

- router-link 方式传递

```
<router-link :to="{name: 'UserProfile', params: {id: 1}}">个人信息</router-link>
```

> 注意： 此时我们将 to 改为了 :to，是为了将这一属性当成对象(Model)使用，注意 router-link 中的 name 属性名称 一定要和 路由中的 name 属性名称 匹配，因为这样 Vue 才能找到对应的路由路径；

- 代码方式传递

```
this.$router.push({ name: 'UserProfile', params: {id: 1}});
```

- 在目标组件中使用以下方式接收参数

```
{{ $route.params.id }}
```

### 使用 props 的方式

- 修改路由配置，主要增加了 props: true 属性

```
{path: '/user/profile/:id', name:'UserProfile', component: UserProfile, props: true}
```

- router-link 方式传递

```
<router-link :to="{name: 'UserProfile', params: {id: 1}}">个人信息</router-link>
```

- 代码方式传递

```
this.$router.push({ name: 'UserProfile', params: {id: 1}});
```

- 接收参数，为目标组件增加 props 属性

```
export default {
  props: ['id'],
  name: "UserProfile"
}
```

- 模板中使用接收参数

```
{{ id }}
```

### 重定向

重定向的意思大家都明白，但 Vue 中的重定向是作用在路径不同但组件相同的情况下

### 修改路由配置

```
    {
      path: '/main',
      name: 'Main',
      component: Main
    },
    {
      path: '/goHome',
      redirect: '/main'
    }
```

说明：这里定义了两个路径，一个是 /main ，一个是 /goHome，其中 /goHome 重定向到了 /main 路径，由此可以看出重定向不需要定义组件；

### 重定向到组件

设置对应路径即可

```
<router-link to="/goHome">回到首页</router-link>
```

### 带参数的重定向

- 修改路由配置

```
    {
      // 首页
      path: '/main/:username',
      name: 'Main',
      component: Main
    },
    {
      path: '/goHome/:username',
      redirect: '/main/:username'
    }
```

- 重定向到组件

```
<router-link to="/goHome/Lusifer">回到首页</router-link>
```
