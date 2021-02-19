# Vue初始化流程

### 生成一个vue实例的流程

```js
// 柯里化方法 - 使用渲染器 render, 返回能实例化一个vue项目的方法 -- createApp
const createAppApi = render => {
  return function createApp(options) {}
}

// 「渲染器」的生成器
const createRenderer = options => {
  const render = (vnode, container) = {}
  return {
    render,
    // 使用渲染器 render, 生成一个vue项目实例
    createApp: createAppApi(render)
  }
}

// 用来渲染的渲染器
const renderer = createRenderer({})

// Vue对象
const Vue = {
  createApp(options) {
    return renderer.createApp(options)
  }
}

// 使用Vue
vue.createApp({
  data() {
    return {
      a: 10,
      b: 20
    }
  },
  mounted() {},
  methods: {},
}).mount('#app')
```