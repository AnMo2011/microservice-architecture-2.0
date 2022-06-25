### 什么是双向数据绑定

Vue.js 是一个 MVVM 框架，即数据双向绑定，即当数据发生变化的时候，视图也就发生变化，当视图发生变化的时候，数据也会跟着同步变化。这也算是 Vue.js 的精髓之处了。值得注意的是，**我们所说的数据双向绑定**，**一定是对于 UI 控件来说的**，非 UI 控件不会涉及到数据双向绑定。单向数据绑定是使用状态管理工具的前提。如果我们使用 **vuex**，那么数据流也是单项的，这时就会和双向数据绑定有冲突。

### 为什么要实现数据的双向绑定

在 Vue.js 中，如果使用 **vuex**，实际上数据还是单向的，之所以说是数据双向绑定，这是用的 UI 控件来说，对于我们处理表单，Vue.js 的双向数据绑定用起来就特别舒服了。即两者并不互斥，在全局性数据流使用单项，方便跟踪；局部性数据流使用双向，简单易操作。

### 在表单中使用双向数据绑定

你可以用 v-model 指令在表单 input、textarea 及 select 元素上创建双向数据绑定。它会根据控件类型自动选取正确的方法来更新元素。尽管有些神奇，但 v-model 本质上不过是语法糖。它负责监听用户的输入事件以更新数据，并对一些极端场景进行一些特殊处理。

> **注意**： **v-model** 会忽略所有表单元素的 **value**、**checked**、**selected** 特性的初始值而总是将 Vue 实例的数据作为数据来源。你应该通过 JavaScript 在组件的 **data** 选项中声明初始值。

### 单行文本

```
<div id="vue">
    单行文本：<input type="text" v-model="message" />  单行文本是：{{message}}
</div>
```

```
<script type="text/javascript">
    var vm = new Vue({
        el: '#vue',
        data: {
            message: "Hello Vue"
        }
    });
</script>
```

![](../img/10-00000022.png)

### 多行文本

```
<div id="vue">
    多行文本：<textarea v-model="message"></textarea>  多行文本是：{{message}}
</div>
```

```
<script type="text/javascript">
    var vm = new Vue({
        el: '#vue',
        data: {
            message: "Hello Textarea"
        }
    });
</script>
```

![](../img/10-00000023.png)

### 单复选框

```
<div id="vue">
    单复选框：<input type="checkbox" id="checkbox" v-model="checked">  <label for="checkbox">{{ checked }}</label>
</div>
```

```
<script type="text/javascript">
    var vm = new Vue({
        el: '#vue',
        data: {
            checked: false
        }
    });
</script>
```

![](../img/10-00000024.png)

### 多复选框

```
<div id="vue">
    多复选框：
    <input type="checkbox" id="jack" value="Jack" v-model="checkedNames">
    <label for="jack">Jack</label>
    <input type="checkbox" id="john" value="John" v-model="checkedNames">
    <label for="john">John</label>
    <input type="checkbox" id="mike" value="Mike" v-model="checkedNames">
    <label for="mike">Mike</label>
    <span>选中的值: {{ checkedNames }}</span>
</div>
```

```
<script type="text/javascript">
    var vm = new Vue({
        el: '#vue',
        data: {
            checkedNames: []
        }
    });
</script>
```

![](../img/10-00000025.png)

### 单选按钮

```
<div id="vue">
    单选按钮：
    <input type="radio" id="one" value="One" v-model="picked">
    <label for="one">One</label>
    <input type="radio" id="two" value="Two" v-model="picked">
    <label for="two">Two</label>
    <span>选中的值: {{ picked }}</span>
</div>
```

```
<script type="text/javascript">
    var vm = new Vue({
        el: '#vue',
        data: {
            picked: ''
        }
    });
</script>
```

![](../img/10-00000026.png)

### 下拉框

```
<div id="vue">
    下拉框：
    <select v-model="selected">
        <option disabled value="">请选择</option>
        <option>A</option>
        <option>B</option>
        <option>C</option>
    </select>
    <span>选中的值: {{ selected }}</span>
</div>
```

```
<script type="text/javascript">
    var vm = new Vue({
        el: '#vue',
        data: {
            selected: ''
        }
    });
</script>
```

![](../img/10-00000027.png)

> **注意**： 如果 **v-model** 表达式的初始值未能匹配任何选项，**select** 元素将被渲染为“未选中”状态。在 iOS 中，这会使用户无法选择第一个选项。因为这样的情况下，iOS 不会触发 change 事件。因此，更推荐像上面这样提供一个值为空的禁用选项。
