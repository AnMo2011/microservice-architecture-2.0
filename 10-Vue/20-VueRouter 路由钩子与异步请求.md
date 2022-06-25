### 路由中的钩子函数

- **beforeRouteEnter**：在进入路由前执行

- **beforeRouteLeave**：在离开路由前执行

案例代码如下：

```js
  export default {
    props: ['id'],
    name: "UserProfile",
    beforeRouteEnter: (to, from, next) => {
      console.log("准备进入个人信息页");
      next();
    },
    beforeRouteLeave: (to, from, next) => {
      console.log("准备离开个人信息页");
      next();
    }
  }
```

参数说明：

- to：路由将要跳转的路径信息

- from：路径跳转前的路径信息

- next：路由的控制参数

    - next() 跳入下一个页面
    
    - next('/path') 改变路由的跳转方向，使其跳到另一个路由
    
    - next(false) 返回原来的页面
    
    - next((vm)=>{}) **仅在 beforeRouteEnter 中可用，vm 是组件实例**

### 在钩子函数中使用异步请求

安装 Axios

    npm install axios -s --registry=https://registry.npm.taobao.org

引用 Axios

    import axios from 'axios'
    Vue.prototype.axios = axios;

在 beforeRouteEnter 中进行异步请求，案例代码如下：

```js
      export default {
        props: ['id'],
        name: "UserProfile",
        beforeRouteEnter: (to, from, next) => {
          console.log("准备进入个人信息页");
          // 注意，一定要在 next 中请求，因为该方法调用时 Vue 实例还没有创建，此时无法获取到 this 对象，在这里使用官方提供的回调函数拿到当前实例
          next(vm => {
            vm.getData();
          });
        },
        beforeRouteLeave: (to, from, next) => {
          console.log("准备离开个人信息页");
          next();
        },
        methods: {
          getData: function () {
            this.axios({
              method: 'get',
              url: 'http://localhost:8080/static/data.json'
            }).then(function (repos) {
              console.log(repos);
            }).catch(function (error) {
              console.log(error);
            });
          }
        }
      }
```