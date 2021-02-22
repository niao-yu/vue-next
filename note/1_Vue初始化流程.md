# Vue初始化流程

调用 `vue.createApp` 后的流程：
- vue.createApp === [packages/runtime-dom/src/index.ts].createApp 方法，这个方法中会执行 `const app = ensureRenderer().createApp(...args)`，来生成待返回的 vue 实例 app。
- ensureRenderer() === [packages/runtime-core/src/renderer.ts].createRenderer(rendererOptions) === baseCreateRenderer(options) === `{ render, hydrate, createApp: createAppAPI(render, hydrate) }`，其中的 render 是在当前方法中生成的，当前方法的主要任务也就是生成 render 函数。
- 其中，对象中那个关键方法 createApp === createAppAPI(render, hydrate) === [packages/runtime-core/src/apiCreateApp.ts].createAppAPI(render, hydrate)，最终会返回一个名为 createApp 的方法，它是创建 vue 实例的方法。
- 所以，去掉中间步骤，vue.createApp 的返回值，就是 createApp「创建 vue 实例的方法」的返回值，最终结果就是 app「vue 实例」。
- vue 官方之所以把这个步骤做的这么复杂，是为了让开发者方便进行二次开发。

配置对象 rendererOptions 声明于文件 [packages/runtime-core/src/renderer.ts]，此文件有一行：`const rendererOptions = extend({ patchProp, forcePatchProp }, nodeOps)`

rendererOptions 提供了操作 dom 的方法，用于渲染真实 dom 过程中，所以此配置是渲染 dom 时的关键，这些方法最终将被使用于 baseCreateRenderer 中的 patch 方法中。

开发者二次开发或者定制 vue 时，这些操作dom的方法就是主要自定义的方向。

通过上面的流程，可以发现以下：
- [packages/runtime-core/src/renderer.ts].baseCreateRenderer(options) 主要开发了 render、patch 函数。
  - render 函数的作用：传入「虚拟dom」和「容器元素」，调用 patch 方法。
  - patch 方法的作用：接受新旧两个节点对象，和其他多个参数，处理后最终调用方法渲染为真实dom。
- [packages/runtime-core/src/apiCreateApp.ts].createAppAPI(render, hydrate) 主要开发了 app 这个 vue 实例对象。

注意我们使用 vue3 时的语句：
```js
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
在 vue.createApp() 后，我们会执行 mount('#app') 来挂载，而这个 mount 方法，明显就是属于 app 对象的。

所以，当开始使用 vue 框架，实例化出一个 vue 实例后，第一个执行就是 mount 方法，这个方法写在 [packages/runtime-core/src/apiCreateApp.ts] 中。
