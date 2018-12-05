开发单页面应用的同学一定对 `路由嵌套`，`子路由` 的概念不会陌生，我们通常使用子路由让页面加载不同的内容，展示不同的效果。这里贴一个 [Vue官方路由的demo](https://jsfiddle.net/yyx990803/L7hscd8h/)，我们拿这个小demo来做例子

```
<div id="app">
  <p>
    <router-link to="/user/detail">detail</router-link>
    <router-link to="/user/profile">profile</router-link>
    <router-link to="/user/posts">posts</router-link>
  </p>
  <router-view></router-view>
</div>
```
这是官方的路由demo代码，我们在上面稍微改动了一下，功能上就是点击对应的链接来显示相应的组件内容。下面我们使用Vue slot-scope来实现类似的效果，首先把app里面的内容也就是 **innerHTML** 拿出来放入一个组件中去，我们暂且叫这个组件为 **parent**。去掉代码里`router-link`和`router-view`的引用，下面的代码是 **parent** 组件模板部分。
```
<div>
  <p @click="clickHandler">
    <button data-component-name="detail">detail</button>
    <button data-component-name="profile">profile</button>
    <button data-component-name="posts">posts</button>
  </p>
  <slot></slot>
</div>
```
我们使用`data-component-name` html5自定义属性来定义要渲染的组件名，这里就是路由映射的组件名。同时把里面的`router-view`去掉换成了`slot`，还有一个地方我们也改动了一下，那就是在原来的`p`标签上我们增加了一个事件处理，这个地方我们是通过事件捕获的机制来处理对应 **p** 标签下面子元素 `button`的点击事件。点击 **button** 我们来显示不同的页面内容，下面来编写这部分js代码来实现router-link的相关功能
```
export default {
    data () {
        return {
            componentName: ''
        }
    },
    methods: {
        clickHandler (e) {
            const elm = e.target
            if (elm.tagName.toLocaleLowerCase() === 'button') { // 处理button按钮的点击事件
                this.componentName = elm.dataset['component-name'] || ''
            }
        }
    }
}
```
我们定义了一个 **componentName** 来接收 **button** 点击时切换的组件名称。基本的代码我们现在已经写完了，那么接下来就是要解决点击按钮是如果根据不同的 **componentName** 加载不同的组件。接下来就是slot-scope登场的时刻了。

我们在 **parent** 组件模板 **slot-scope** 上增加一个componentName，这样我们就可以在parent的子组件中去访问要显示的组件名称。如果parent组件没有实现子组件那么会有一个友好的提示
```
<slot :componentName="componentName">parent组件缺少默认slot</slot>
```
然后我们在子组件引用父组件
```
<parent>
    <div slot-scope="{componentName}">
        {{compnentName}}
        <component :is="componentName"></component>
    </div>
</parent>
```
这里我们使用作用域插槽来获取到要渲染的组件。并且使用 **component** 语法来动态渲染组件。这样，我们就完全实现了点击按钮显示不同组件的功能了
