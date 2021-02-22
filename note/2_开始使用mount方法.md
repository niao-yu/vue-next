# 开始使用mount方法

## mount 方法

当开始使用 vue 框架，实例化出一个 vue 实例后，第一个执行就是 mount 方法，这个方法写在 [packages/runtime-core/src/apiCreateApp.ts] 中。

mount 方法：
```js
mount(rootContainer: HostElement, isHydrate?: boolean): any {
  if (!isMounted) {
    // 去生成虚拟dom
    const vnode = createVNode(
      rootComponent as ConcreteComponent,
      rootProps
    )
    // store app context on the root VNode.
    // this will be set on the root instance on initial mount.
    vnode.appContext = context

    // HMR root reload
    if (__DEV__) {
      context.reload = () => {
        render(cloneVNode(vnode), rootContainer)
      }
    }

    if (isHydrate && hydrate) {
      hydrate(vnode as VNode<Node, Element>, rootContainer as any)
    } else {
      // 把虚拟dom渲染为dom
      render(vnode, rootContainer)
    }
    isMounted = true
    app._container = rootContainer
    // for devtools and telemetry
    ;(rootContainer as any).__vue_app__ = app

    if (__DEV__ || __FEATURE_PROD_DEVTOOLS__) {
      devtoolsInitApp(app, version)
    }

    return vnode.component!.proxy
  } else if (__DEV__) {
    warn(
      `App has already been mounted.\n` +
        `If you want to remount the same app, move your app creation logic ` +
        `into a factory function and create fresh app instances for each ` +
        `mount - e.g. \`const createMyApp = () => createApp(App)\``
    )
  }
}
```

mount关键的两个方法：
- createVNode：根据把初始化 vue 时传入的参数，生成虚拟dom -- vnode。
- render：把生成的 vnode 渲染到容器元素中。

### createVNode - 生成虚拟dom

createVNode 方法位于：[packages/runtime-core/src/vnode.ts] 文件中，正式环境下，直接等同于 _createVNode 方法。

方法最后会拼装出一个虚拟dom vnode 对象，代码如下，可见大多字段的值还都是null。

**注意，这个虚拟dom vnode，是包含了这个虚拟dom 的一些其他信息的，当真正要拿它去渲染真实 dom 时，需要先转换为 虚拟dom树(subTree)。**

```js
const vnode: VNode = {
  __v_isVNode: true,
  [ReactiveFlags.SKIP]: true,
  type,
  props,
  key: props && normalizeKey(props),
  ref: props && normalizeRef(props),
  scopeId: currentScopeId,
  children: null,
  component: null,
  suspense: null,
  ssContent: null,
  ssFallback: null,
  dirs: null,
  transition: null,
  el: null,
  anchor: null,
  target: null,
  targetAnchor: null,
  staticCount: 0,
  shapeFlag,
  patchFlag,
  dynamicProps,
  dynamicChildren: null,
  appContext: null
}
```

得到 vnode 之后，执行 normalizeChildren(vnode, children)，这个方法，貌似只是根据 vnode 的内容的类型，赋值一下 shapeFlag 字段，并把 children 的值修改一下类型。

## render - 渲染虚拟dom

render 方法来回传递，归根结底声明于文件 [packages/runtime-core/src/renderer.ts] 中的 baseCreateRenderer(options) 方法中。

```js
/**
 * render 函数，对比并更新虚拟 dom，最后渲染成真实dom
 * @param vnode 新的虚拟dom
 * @param container 容器元素，其中包含当前容器的旧虚拟dom
 */
const render: RootRenderFunction = (vnode, container) => {
  if (vnode == null) {
    if (container._vnode) {
      // 没有新的虚拟dom，直接卸载旧节点即可
      unmount(container._vnode, null, null, true)
    }
  } else {
    // 对比并更新
    patch(container._vnode || null, vnode, container)
  }
  flushPostFlushCbs()
  container._vnode = vnode
}
```

可见：
- 新虚拟 dom 为 null，直接从 container 中卸载虚拟 dom
- 新虚拟 dom 存在，调用 patch 方法，进行对比并更新

### patch 方法内容

patch 声明于文件 [packages/runtime-core/src/renderer.ts] 中的 baseCreateRenderer(options) 方法中。

patch 中使用的操作 dom 的方法，取自于 baseCreateRenderer 接受的参数 options：
```js
const {
  insert: hostInsert,
  remove: hostRemove,
  patchProp: hostPatchProp,
  forcePatchProp: hostForcePatchProp,
  createElement: hostCreateElement,
  createText: hostCreateText,
  createComment: hostCreateComment,
  setText: hostSetText,
  setElementText: hostSetElementText,
  parentNode: hostParentNode,
  nextSibling: hostNextSibling,
  setScopeId: hostSetScopeId = NOOP,
  cloneNode: hostCloneNode,
  insertStaticContent: hostInsertStaticContent
} = options
```

options 来自于上游传入的配置对像 rendererOptions，rendererOptions 声明于文件 [packages/runtime-core/src/renderer.ts]，此文件有一行：`const rendererOptions = extend({ patchProp, forcePatchProp }, nodeOps)` 。

其继承于 nodeOps 对象，nodeOps 声明于 [packages/runtime-dom/src/nodeOps.ts]。

patch 方法的作用：接受新旧两个节点对象，和其他多个参数，判断新节点的类型(虚拟dom或虚拟dom树)，或者另一个维度的类型(文字、组件、静态资源等)，针对不同类型的节点，来声明不同的方法，调用上面 options 中取出来的 dom 操作方法，来进行差异化处理。

patch 接受一系列参数，其中有旧节点对象和新节点对象，注意这个节点对象，可能是虚拟dom、虚拟dom树等等。

会根据新旧节点的 type 的值来判断类型（以下类型声明于此文件中[packages/runtime-core/src/vnode.ts]）：
- 新 type === Text：文字，调用 processText 方法
- 新 type === Comment：组件，调用 processCommentNode 方法
- 新 type === Static：静态资源，调用 mountStaticNode 或 patchStaticNode 方法
- 新 type === Fragment：html片段，调用 processFragment 方法

如果都不属于，会再根据新节点的 shapeFlag 的值来继续判断（以下类型声明于此文件中[packages/shared/src/shapeFlags.ts]）：
- 新 shapeFlag === ELEMENT：是元素，调用 processElement 方法
- 新 shapeFlag === COMPONENT：组件，调用 processComponent 方法
- 新 shapeFlag === TELEPORT：...
- 新 shapeFlag === SUSPENSE：...

以上执行完成，最后再执行 setRef 方法，设置或删除 ref，patch 方法完成任务。

### 组件类型的虚拟dom 处理 processComponent

程序最初执行时，依据 新节点的 type 无法判断，最后根据 新节点的 shapeFlag 判断出其属于组件，会去执行 processComponent 方法。

其中关键步骤，如果判断得出，新虚拟dom是组件，则会执行 processComponent 方法，里面会再判断该组件是否已经挂载：
- 未挂载：调用 mountComponent 进行挂载
- 已挂载：调用 updateComponent 更新组件

### mountComponent 方法初始化组件

在 mountComponent 方法中，需要初始化组件，也就是把开发者传入的包含 data、methods、mounted、watch...等字段的对象，整理监听，每个字段都处理到合适的位置。

- 执行 [packages/runtime-core/src/component.ts].createComponentInstance，生成组件实例 instance
- 调用 [packages/runtime-core/src/component.ts].setupComponent 方法个组件实例添加功能，调用 initProps 和 initSlots。
- 调用 setupRenderEffect，给这个组件的 update 方法绑定上一个发布者
  - setupRenderEffect 方法内部执行 `instance.update = effect(function componentEffect() {...}, effectOptions)`，也就是调用 effect 方法，并传过去一个更新时的回调函数 componentEffect 和配置对象 effectOptions。
  - 先说，effect 方法，这个方法在文件 [packages/reactivity/src/effect.ts] 中。
    - 上面传给 effect 方法的回调函数 componentEffect，在 effect 中使用 fn 这个变量接收，配置对像 effectOptions 使用变量 options 接收。
    - effect 方法调用 createReactiveEffect 方法来创建发布者对象，把 fn 和 options 传过去。
      - 声明发布者，本身为一个可执行方法，同时把它作为对象，设置所需字段，这个发布者方法，最终会返回到顶层，赋值给 `instance.update`，所以，当该组件需要更新时，就会执行这个发布者方法。
      - 发布者方法的内部内容并不多。。。反正是会执行一次传过来的 fn 方法。
      - 最终，把发布者方法返回给上层的 effect 方法。
    - effect 方法拿到发布者方法，回先判断配置对象 options 中关于执行时机的字段 lazy，如果为 false，则马上执行一次发布者方法，最后，把发布者方法返回，赋值给 instance.update。
  - 回调函数 componentEffect，根据上面 effect 方法的研究，会发现，当绑定完成和执行组件 update 方法时，componentEffect 都会执行一次。
    - 判断当前组件是否已挂载，如果已经挂载，那就是各种判断对比，取旧的虚拟dom树，生成新的虚拟dom树，去调用 patch 方法进行对比渲染，同时出发多个钩子函数。
    - 如果尚未挂载，则生成虚拟dom树，再在一次去调用 patch 方法进行更新。


这里需要说明一个对象，在文件 [packages/reactivity/src/effect.ts] 中，文件顶部声明了一个对象 `const targetMap = new WeakMap<any, KeyToDepMap>()`。

targetMap 即是存储所有依赖的对象，
const targetMap = new WeakMap<any, KeyToDepMap>()


effectStack：const effectStack: ReactiveEffect[] = []

trackStack：const trackStack: boolean[] = []
