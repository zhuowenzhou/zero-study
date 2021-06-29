# vue代码优化

1. ## v-if 和 v-show

**v-if** 是 真正 的条件渲染，因为它会确保在切换过程中条件块内的事件监听器和子组件适当地被销毁和重建；也是惰性的：如果在初始渲染时条件为假，则什么也不做——直到条件第一次变为真时，才会开始渲染条件块。

**v-show** 就简单得多， 不管初始条件是什么，元素总是会被渲染，并且只是简单地基于 CSS 的 display 属性进行切换。
所以，v-if 适用于在运行时很少改变条件，不需要频繁切换条件的场景；v-show 则适用于需要非常频繁切换条件的场景。


2. ## compted 和 watch

**computed**： 是计算属性，依赖其它属性值，并且 computed 的值有缓存，只有它依赖的属性值发生改变，下一次获取 computed 的值时才会重新计算 computed  的值；

**watch**： 更多的是「观察」的作用，类似于某些数据的监听回调 ，每当监听的数据变化时都会执行回调进行后续操作；

运用场景：
* 当我们需要进行数值计算，并且依赖于其它数据时，应该使用 computed，因为可以利用 computed 的缓存特性，避免每次获取值时，都要重新计算；
* 当我们需要在数据变化时执行异步或开销较大的操作时，应该使用 watch，使用 watch 选项允许我们执行异步操作 ( 访问一个 API )，限制我们执行该操作的频率，并在我们得到最终结果前，设置中间状态。这些都是计算属性无法做到的。

3. ## v-for 遍历必须为 item 添加 key，且避免同时使用 v-if
v-for 比 v-if 优先级高，如果每一次都需要遍历整个数组，将会影响速度，尤其是当之需要渲染很小一部分的时候，必要情况下应该替换成 computed 属性。

4. ## 长列表性能优化

通过 Object.freeze 方法来冻结一个对象，使数据劫持不生效，常用于纯展示类的列表数据。
```javascript
export default {
  data: () => ({
    employee: []
  }),
  async created() {
    const employee = await axios.get("/api/employee");
    this.employee = Object.freeze(employee);
  }
};
```

5. ## 事件的注册和销毁
Vue 组件销毁时，会自动清理它与其它实例的连接，解绑它的全部指令及事件监听器，但是仅限于组件本身的事件。 如果在 js 内使用 addEventListene 等方式是不会自动销毁的，我们需要在组件销毁时手动移除这些事件的监听，以免造成内存泄露。

```javascript
created() {
  addEventListener('click', this.handleClick, false)
},
beforeDestroy() {
  removeEventListener('click', this.handleClick, false)
}
```

6. ## 图片资源懒加载
通过插件 vue-lazyload  实现图片出现在视窗范围在进行加载

```javascript
// scripts
import VueLazyload from 'vue-lazyload'
Vue.use(VueLazyload, {
preLoad: 1.3,
error: '/static/error.png',
loading: '/static/loading.gif',
attempt: 1
});

//template
<img v-lazy="/static/img/1.png">

```

7. ## 路由懒加载

```javascript
const Home = () => import('./Home.vue')
const router = new VueRouter({
  routes: [
    { path: '/home', component: Home }
  ]
})

```

8. ## 第三方插件的按需引入
借助 babel-plugin-component ，然后可以只引入需要的组件，以达到减小项目体积的目的
1. 安装 babel-plugin-component

```bash
npm install babel-plugin-component -D
```
2. 修改 .babelrc 
``` javascript
{
  "presets": [["es2015", { "modules": false }]],
  "plugins": [
    [
      "component",
      {
        "libraryName": "element-ui",
        "styleLibraryName": "theme-chalk"
      }
    ]
  ]
}
```

3. 使用
``` javascript
import Vue from 'vue';
import { Button, Select } from 'element-ui';
Vue.use(Button)
Vue.use(Select)
```

9. ## 优化无限列表性能
如果你的应用存在非常长或者无限滚动的列表，那么需要采用 窗口化 的技术来优化性能，只需要渲染少部分区域的内容，减少重新渲染组件和创建 dom 节点的时间。 你可以参考以下开源项目 [vue-virtual-scroll-list](https://github.com/tangbc/vue-virtual-scroll-list) 和 [vue-virtual-scroller](https://github.com/Akryum/vue-virtual-scroller)  来优化这种无限列表的场景的。


10. ## 服务端渲染 SSR or 预渲染

服务端渲染是指 Vue 在客户端将标签渲染成的整个 html 片段的工作在服务端完成，服务端形成的 html 片段直接返回给客户端这个过程就叫做服务端渲染。

1. 优点
 * 更好的 SEO
 * 更快的内容到达时间（首屏加载更快）
2. 缺点
 * 更多的开发条件限制： 例如服务端渲染只支持 beforCreate 和 created 两个钩子函数，这会导致一些外部扩展库需要特殊处理，才能在服务端渲染应用程序中运行；
 * 需要部署在nodejs环境
 * 更多的服务器负载：在 Node.js 中渲染完整的应用程序，显然会比仅仅提供静态文件的 server 更加大量占用CPU 资源，因此如果你预料在高流量环境下使用，请准备相应的服务器负载，并明智地采用缓存策略

如果你的 Vue 项目只需改善少数营销页面（例如  /， /about， /contact 等）的 SEO，那么你可能需要预渲染，在构建时 (build time) 简单地生成针对特定路由的静态 HTML 文件。优点是设置预渲染更简单，并可以将你的前端作为一个完全静态的站点，具体你可以使用 prerender-spa-plugin 就可以轻松地添加预渲染 。
