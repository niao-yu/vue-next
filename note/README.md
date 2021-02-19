# 概括说明

此项目来fock自以下vue官方仓库：`https://github.com/vuejs/vue-next.git`

## 开发说明

安装依赖：`yarn --ignore-scripts`
> 忽略一些不必要的依赖

修改`package.json`文件的`dev`指令为：`"dev": "node scripts/dev.js --sourcemap",`
> 打包时带上`sourcemap`，方便调试

## 主要目录

代码仓库中有个 packages 目录，里面是 Vue 3 的主要功能的实现，包括：
- compiler-core 目录：平台无关的编译器. 它既包含可扩展的基础功能，也包含所有平台无关的插件。
- compiler-dom 目录：针对浏览器而写的编译器。
- reactivity 目录：数据响应式系统，这是一个单独的系统，可以与任何框架配合使用。
- runtime-core 目录：与平台无关的运行时。其实现的功能有虚拟 DOM 渲染器、Vue 组件和 Vue 的各种API，我们可以利用这个 runtime 实现针对某个具体平台的高阶 runtime，比如自定义渲染器。
- runtime-dom 目录：针对浏览器的 runtime。其功能包括处理原生 DOM API、DOM 事件和 DOM 属性等。
  - 是使用 vue 的入口，createApp 这个方法，就是从这个文件夹中声明的。
- runtime-test 目录：一个专门为了测试而写的轻量级 runtime。由于这个 rumtime 「渲染」出的 DOM 树其实是一个 JS 对象，所以这个 runtime 可以用在所有 JS 环境里。你可以用它来测试渲染是否正确。它还可以用于序列化 DOM、触发 DOM 事件，以及记录某次更新中的 DOM 操作。
- server-renderer 目录：用于 SSR。尚未实现。
- shared 目录：没有暴露任何 API，主要包含了一些平台无关的内部帮助方法。
- vue 目录：用于构建「完整构建」版本，引用了上面提到的 runtime 和 compiler


## 参考网站

- [Vue 3 源码导读](https://juejin.cn/post/6844903957421096967#heading-8)