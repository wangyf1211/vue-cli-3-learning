# Vue源码学习
## 项目文件结构介绍

    ├── dist ---------------------------------------- 构建后文件的输出目录
    ├── examples ------------------------------------ 存放一些使用Vue开发的应用案例
    ├── flow ---------------------------------------- 类型声明，使用开源项目 [Flow](https://flowtype.org/)
    ├── package.json -------------------------------- package.json
    ├── test ---------------------------------------- 包含所有测试文件
    ├── src ----------------------------------------- 源码
    │   ├── platforms ------------------------------- 包含了不同平台特有的相关代码
    |   |   ├── web --------------------------------- web平台下的构建配置文件
    │   │   |   ├── entry-runtime.js ---------------- 运行时构建的入口，输出 dist/vue.common.js 文件，不包含模板(template)到render函数的编译器，所以不支持 `template` 选项，我们使用vue默认导出的就是这个运行时的版本。大家使用的时候要注意
    │   │   |   ├── entry-runtime-with-compiler.js -- 独立构建版本的入口，输出 dist/vue.js，它包含模板(template)到render函数的编译器
    │   │   |   ├── entry-compiler.js --------------- vue-template-compiler 包的入口文件
    │   │   |   ├── entry-server-renderer.js -------- vue-server-renderer 包的入口文件
    |   |   ├── weex -------------------------------- weex平台下的构建配置文件
    │   ├── compiler -------------------------------- 编译器代码的存放目录，将 template 编译为 render 函数
    │   │   ├── parser ------------------------------ 存放将模板字符串转换成元素抽象语法树的代码
    │   │   ├── codegen ----------------------------- 存放从抽象语法树(AST)生成render函数的代码
    │   │   ├── optimizer.js ------------------------ 分析静态树，优化vdom渲染
    │   ├── core ------------------------------------ 存放通用的，平台无关的代码
    │   │   ├── observer ---------------------------- 反应系统，包含数据观测的核心代码
    │   │   ├── vdom -------------------------------- 包含虚拟DOM创建(creation)和打补丁(patching)的代码
    │   │   ├── instance ---------------------------- 包含Vue构造函数设计相关的代码
    │   │   ├── global-api -------------------------- 包含给Vue构造函数挂载全局方法(静态方法)或属性的代码
    │   │   ├── components -------------------------- 包含抽象出来的通用组件 KeepAlive
    │   ├── server ---------------------------------- 包含服务端渲染(server-side rendering)的相关代码
    │   ├── sfc ------------------------------------- 包含单文件组件(.vue文件)的解析逻辑，用于vue-template-compiler包
    │   ├── shared ---------------------------------- 包含整个代码库通用的代码

## Vue的构造函数
运行<code>npm run dev</code>之后会输出<code>dist/vue.js</code>，看下这条命令主要做了哪些工作？
查看**package.json**文件，对应<code>scripts</code>中的<code>dev</code>部分，可以看到如下
```
"dev": "rollup -w -c scripts/config.js --environment TARGET:web-full-dev"
```
<code>rollup</code>是JavaScript模块打包器，下面主要看下<code>scripts/config.js</code>文件

**scripts/config.js**文件中
主要为<code>genConfig</code>函数，返回一个config对象，而这个config对象就是Rollup的配置对象

这里为<code>genConfig(builds[process.env.TARGET])</code>解析之后为
```
module.exports = genConfig({
    entry: resolve('web/entry-runtime-with-compiler.js'),
    dest: resolve('dist/vue.js'),
    format: 'umd',
    env: 'development',
    alias: { he: './entity-decoder' },
    banner
})
```
跟着线索，继续看<code>web/entry-runtime-with-compiler.js</code>文件，打开文件，看到如下
```
import Vue from './runtime/index'
```
继续查看<code>web/runtime/index.js</code>，打开文件，看到如下
```
import Vue from 'core/index'
```
继续查看<code>core/index.js</code>，打开文件，看到如下
```
import Vue from './instance/index'
```
继续查看<code>core/instance/index.js</code>文件，看到如下
```
import { initMixin } from './init'
import { stateMixin } from './state'
import { renderMixin } from './render'
import { eventsMixin } from './events'
import { lifecycleMixin } from './lifecycle'
import { warn } from '../util/index'

function Vue (options) {
  if (process.env.NODE_ENV !== 'production' &&
    !(this instanceof Vue)
  ) {
    warn('Vue is a constructor and should be called with the `new` keyword')
  }
  this._init(options)
}

initMixin(Vue)
stateMixin(Vue)
eventsMixin(Vue)
lifecycleMixin(Vue)
renderMixin(Vue)

export default Vue
```
终于看到了构造函数的定义，以构造函数为参数，调用了五个方法又导出Vue，可以看下这五个方法<code>initMixin</code>、<code>stateMixin</code>、<code>eventsMixin</code>、<code>lifecycleMixin</code>、<code>renderMixin</code>
```
// initMixin(Vue)    src/core/instance/init.js **************************************************
Vue.prototype._init = function (options?: Object) {}

// stateMixin(Vue)    src/core/instance/state.js **************************************************
Object.defineProperty(Vue.prototype, '$data', dataDef)
Object.defineProperty(Vue.prototype, '$props', propsDef)
Vue.prototype.$set = set
Vue.prototype.$delete = del
Vue.prototype.$watch = function(){}

// renderMixin(Vue)    src/core/instance/render.js **************************************************
// install runtime convenience helpers
installRenderHelpers(Vue.prototype) 此方法在core/instance/render-helpers/index.js文件中
<!-- 主要为以下
  target._o = markOnce
  target._n = toNumber
  target._s = toString
  target._l = renderList
  target._t = renderSlot
  target._q = looseEqual
  target._i = looseIndexOf
  target._m = renderStatic
  target._f = resolveFilter
  target._k = checkKeyCodes
  target._b = bindObjectProps
  target._v = createTextVNode
  target._e = createEmptyVNode
  target._u = resolveScopedSlots
  target._g = bindObjectListeners
  target._d = bindDynamicKeys
  target._p = prependModifier -->
Vue.prototype.$nextTick = function (fn: Function) {}
Vue.prototype._render = function (): VNode {}

// eventsMixin(Vue)    src/core/instance/events.js **************************************************
Vue.prototype.$on = function (event: string, fn: Function): Component {}
Vue.prototype.$once = function (event: string, fn: Function): Component {}
Vue.prototype.$off = function (event?: string, fn?: Function): Component {}
Vue.prototype.$emit = function (event: string): Component {}

// lifecycleMixin(Vue)    src/core/instance/lifecycle.js **************************************************
Vue.prototype._update = function (vnode: VNode, hydrating?: boolean) {}
Vue.prototype.$forceUpdate = function(){}
Vue.prototype.$destroy = function () {}
```
既然找到了Vue的构造函数，还记得我们怎么来的吗？现在进行路线回溯，看看刚刚那几个文件主要都做了啥

首先是<code>core/index.js</code>
```
import Vue from './instance/index'
import { initGlobalAPI } from './global-api/index'
import { isServerRendering } from 'core/util/env'
import { FunctionalRenderContext } from 'core/vdom/create-functional-component'

initGlobalAPI(Vue)

Object.defineProperty(Vue.prototype, '$isServer', {
  get: isServerRendering
})

Object.defineProperty(Vue.prototype, '$ssrContext', {
  get () {
    /* istanbul ignore next */
    return this.$vnode && this.$vnode.ssrContext
  }
})

// expose FunctionalRenderContext for ssr runtime helper installation
Object.defineProperty(Vue, 'FunctionalRenderContext', {
  value: FunctionalRenderContext
})

Vue.version = '__VERSION__'

export default Vue
```
首先对Vue执行<code>initGlobalAPI</code>方法，作用是在 Vue 构造函数上挂载静态属性和方法

接下来是<code>web/runtime/index.js</code>，主要做了三件事
```
1、覆盖 Vue.config 的属性，将其设置为平台特有的一些方法
2、Vue.options.directives 和 Vue.options.components 安装平台特有的指令和组件
3、在 Vue.prototype 上定义 __patch__ 和 $mount
```
经过web/runtime/index.js的作用，<code>Vue</code>为以下：
```
// 安装平台特定的utils
Vue.config.isUnknownElement = isUnknownElement
Vue.config.isReservedTag = isReservedTag
Vue.config.getTagNamespace = getTagNamespace
Vue.config.mustUseProp = mustUseProp
// 安装平台特定的 指令 和 组件
Vue.options = {
    components: {
        KeepAlive,
        Transition,
        TransitionGroup
    },
    directives: {
        model,
        show
    },
    filters: {},
    _base: Vue
}
Vue.prototype.__patch__
Vue.prototype.$mount
```
<code>$mount</code>的逻辑是根据是否是浏览器环境决定要不要query(el)获取元素，然后将el作为参数传递给this._mount()，看下面代码
```
Vue.prototype.$mount = function (
  el?: string | Element,
  hydrating?: boolean
): Component {
  el = el && inBrowser ? query(el) : undefined
  return mountComponent(this, el, hydrating)
}
```

接下来看下<code>web/entry-runtime-with-compiler.js</code>，该文件做了两件事：

    1. 缓存来自 web-runtime.js 文件的 $mount 函数
    const mount = Vue.prototype.$mount
    然后覆盖覆盖了 Vue.prototype.$mount
    2. 在 Vue 上挂载 compile
    Vue.compile = compileToFunctions
    compileToFunctions 函数的作用，就是将模板 template 编译为render函数。

至此，我们算是还原了 Vue 构造函数，总结一下：

    1. Vue.prototype 下的属性和方法的挂载主要是在 src/core/instance 目录中的代码处理的

    2. Vue 下的静态属性和方法的挂载主要是在 src/core/global-api 目录下的代码处理的

    3. web/runtime/index.js 主要是添加web平台特有的配置、组件和指令，web/entry-runtime-with-compiler.js 给Vue的 $mount 方法添加 compiler 编译器，支持 template。

以上基于[HcySunYang](http://hcysun.me/2017/03/03/Vue%E6%BA%90%E7%A0%81%E5%AD%A6%E4%B9%A0/)

```
initLifecycle(vm)
initEvents(vm)
callHook(vm, 'beforeCreate')
initProps(vm)
initMethods(vm)
initData(vm)
initComputed(vm)
initWatch(vm)
callHook(vm, 'created')
initRender(vm)
```
因此created之后才可以操纵DOM，因为created之前都没有渲染真正的DOM元素到文档中

## 通过 initData 看Vue的数据响应系统
Vue的数据响应系统包含三个部分：<code>Observer</code>、<code>Dep</code>、<code>Watcher</code>

有点疑问，一般不都是说Observer Watcher和Compiler吗？
首先看下<code>initData</code>的代码
```
function initData (vm: Component) {
  let data = vm.$options.data
  data = vm._data = typeof data === 'function'
    ? data.call(vm)
    : data || {}
  if (!isPlainObject(data)) {
    data = {}
    process.env.NODE_ENV !== 'production' && warn(
      'data functions should return an object:\n' +
      'https://vuejs.org/v2/guide/components.html#data-Must-Be-a-Function',
      vm
    )
  }
  // proxy data on instance
  const keys = Object.keys(data)
  const props = vm.$options.props
  let i = keys.length
  while (i--) {
    if (props && hasOwn(props, keys[i])) {
      process.env.NODE_ENV !== 'production' && warn(
        `The data property "${keys[i]}" is already declared as a prop. ` +
        `Use prop default value instead.`,
        vm
      )
    } else {
      proxy(vm, keys[i])
    }
  }
  // observe data
  observe(data)
  data.__ob__ && data.__ob__.vmCount++
}
```

首先，先拿到 data 数据：let data = vm.$options.data，大家还记得此时 vm.$options.data 的值应该是通过 mergeOptions 合并处理后的 mergedInstanceDataFn 函数吗？所以在得到 data 后，它又判断了 data 的数据类型是不是 ‘function’，最终的结果是：data 还是我们传入的数据选项的 data，即：

data: {
    a: 1,
    b: [1, 2, 3]
}
然后在实例对象上定义 _data 属性，该属性与 data 是相同的引用。

然后是一个 while 循环，循环的目的是在实例对象上对数据进行代理，这样我们就能通过 this.a 来访问 data.a 了，代码的处理是在 proxy 函数中，该函数非常简单，仅仅是在实例对象上设置与 data 属性同名的访问器属性，然后使用 _data 做数据劫持，如下：
```
function proxy (vm: Component, key: string) {
  if (!isReserved(key)) {
    Object.defineProperty(vm, key, {
      configurable: true,
      enumerable: true,
      get: function proxyGetter () {
        return vm._data[key]
      },
      set: function proxySetter (val) {
        vm._data[key] = val
      }
    })
  }
}
```
做完数据的代理，就正式进入响应系统，

observe(data)
我们说过，数据响应系统主要包含三部分：Observer、Dep、Watcher，代码分别存放在：observer/index.js、observer/dep.js 以及 observer/watcher.js 文件中。

以目前的理解讲述一下数据双向绑定的原理。

首先Observer部分，有两个主要函数defineReactive()和observe()，前者主要对属性利用defineProperty()做数据劫持，而observe()主要是遍历data()对象的所有属性，递归调用，对每个属性都添加访问性属性。

对于Watcher，data[exp]时会获取data.exp所以就会调用到get函数，成功的与observer产生了关联。此时再利用Dep，Dep.target设为当前的watcher对象

Dep为依赖收集器，每一个属性对应一个Dep，在defineReactive中对应一个属性就新建一个dep对象，在get()方法中对当前属性对应的dep进行addSub()操作，添加到Dep类的subs数组中，在set涉及到数据变更时，对Dep中的所有依赖进行notify提醒watcher该更新了。