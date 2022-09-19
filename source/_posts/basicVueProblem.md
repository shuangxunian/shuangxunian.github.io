---
title: Vue高频面试题
excerpt: 此篇是Vue3.0以下的高频面试题，3.0的内容还不是很充分
categories:
- 技术文章
tags:
- vue
---

## 前言
欢迎大家私聊我进行补充~这里只放一些比较简介的问题和答案，想看前端面试题请移步[前端必会面试题](https://shuangxunian.gitee.io/2020/09/08/%E5%89%8D%E7%AB%AF%E5%BF%85%E4%BC%9A/)。

## 为什么操作真实 dom 有性能问题
因为 DOM 是属于渲染引擎中的东西，而 JS 又是 JS 引擎中的东西。当我们通过 JS 操作 DOM 的时候，其实这个操作涉及到了两个线程之间的通信，那么势必会带来一些性能上的损耗。操作 DOM 次数一多，也就等同于一直在进行线程之间的通信，并且操作 DOM 可能还会带来重绘回流的情况，所以也就导致了性能上的问题。

## 为什么虚拟dom会提高性能?
虚拟dom相当于在js和真实dom中间加了一个缓存，利用dom diff算法避免了没有必要的dom操作，从而提高性能。
具体实现步骤如下：
1. 用 JavaScript 对象结构表示 DOM 树的结构；然后用这个树构建一个真正的 DOM 树，插到文档当中
2. 当状态变更的时候，重新构造一棵新的对象树。然后用新的树和旧的树进行比较，记录两棵树差异
3. 把2所记录的差异应用到步骤1所构建的真正的DOM树上，视图就更新了。

## 说一下 MVVM
MVC是后端的分层开发概念；
MVVM是前端视图层的概念，主要关注于视图层分离。MVVM把前端的视图层分为了三部分：Model,View,VM ViewModel
- M: Model 数据层
- V: View 视图层
- VM: ViewModel 视图层和数据层中间的桥，视图层和数据层通信的桥梁

view 层通过事件去绑定 Model 层， Model 层通过数据去绑定 View 层

## Vue 的响应式原理
Vue 内部使用了 Object.defineProperty() 来实现数据响应式，通过这个函数可以监听到 set 和 get 的事件。
1. 首先利用 Object.defineProperty() 给 data 中的属性去设置 set, get 事件
2. 递归的去把 data 中的每一个属性注册为被观察者
3. 解析模板时，在属性的 get 事件中去收集观察者依赖
4. 当属性的值发生改变时，在 set 事件中去通知每一个观察者，做到全部更新

## Vue 的模板如何被渲染成 HTML? 以及渲染过程
1. vue 模板的本质是字符串，利用各种正则，把模板中的属性去变成 js 中的变量，vif,vshow,vfor 等指令变成 js 中的逻辑
2. 模板最终会被转换为 render 函数
3. render 函数执行返回 vNode
4. 使用 vNode 的 path 方法把 vNode 渲染成真实 DOM

## Vue 的整个实现流程
1. 先把模板解析成 render 函数，把模板中的属性去变成 js 中的变量，vif,vshow,vfor 等指令变成 js 中的逻辑
2. 执行 render 函数，在初次渲染执行 render 函数的过程中 绑定属性监听，收集依赖，最终得到 vNode，利用 vNode 的 Path 方法，把 vNode 渲染成真实的 DOM
3. 在属性更新后，重新执行 render 函数，不过这个时候就不需要绑定属性和收集依赖了，最终生成新的 vNode
4. 把新的 vNode 和 旧的 vNode 去做对比，找出真正需要更新的 DOM，渲染

## 什么是 diff 算法，或者是 diff 算法用来做什么
- diff 是linux中的基础命令，可以用来做文件，内容之间的对比
- vNode 中使用 diff 算法是为了找出需要更新的节点，避免造成不必要的更新

## Vuex是什么
vuex 就像一个全局的仓库，公共的状态或者复杂组件交互的状态我们会抽离出来放进里面。

vuex的核心主要包括以下几个部分：
- state：state里面就是存放的我们需要使用的状态，他是单向数据流，在 vue 中不允许直接对他进行修改，而是使用 mutations 去进行修改
- mutations: mutations 就是存放如何更改状态的一些方法
- actions： actions 是来做异步修改的，他可以等待异步结束后，再去使用 commit mutations 去修改状态
- getters: 相当于是 state 的计算属性

使用：
- 获取状态在组件内部 computed 中使用 this.$store.state 得到想要的状态
- 修改的话可在组件中使用 this.$store.commit 方法去修改状态
- 如果在一个组件中，方法，状态使用太多。 可以使用 mapState，mapMutations 辅助函数

## Vue 中组件之间的通信
父子组件：父组件通过 Props 传递子组件，子组件通过 $emit 通知父组件 兄弟组件：可以使用 vuex 全局共享状态，或 eventBus

## 如何解决页面刷新后 Vuex 数据丢失的问题
可以通过插件 vuex-persistedstate 来解决 插件原理：利用 HTML5 的本地存储 + Vuex.Store 的方法，去同步本地和 store 中的数据，做到同步更新

## 前端路由的两种实现原理
两种路由模式分为 hash 模式 和 history 模式

- hash 模式：
hash 模式背后的原理是 onhashchange 事件，可以在window对象上监听这个事件，在hash 变化时，浏览器会记录历史，并且触发事件回调，在 hash 模式中，地址栏中会带一个 #

- history 模式：
前面的 hashchange，你只能改变 # 后面的 url 片段，而 history api 则给了前端完全的自由；history api可以分为两大部分，切换和修改，参考 MDN

使用 history 需要服务端的支持，在hash模式下，前端路由修改的是#中的信息，而浏览器请求时是不带着的，所以没有问题.但是在history下，你可以自由的修改path，当刷新时，如果服务器中没有相应的响应或者资源，会刷出一个404来

## 写 React / Vue 项目时为什么要在列表组件中写 key，其作用是什么？
vue和react都是采用diff算法来对比新旧虚拟节点，从而更新节点。在vue的diff函数中（建议先了解一下diff算法过程）。
在交叉对比中，当新节点跟旧节点头尾交叉对比没有结果时，会根据新节点的key去对比旧节点数组中的key，从而找到相应旧节点（这里对应的是一个key => index 的map映射）。如果没找到就认为是一个新增节点。而如果没有key，那么就会采用遍历查找的方式去找到对应的旧节点。一种一个map映射，另一种是遍历查找。相比而言。map映射的速度更快。
vue部分源码如下：
```javascript
// vue项目  src/core/vdom/patch.js  -488行
// 以下是为了阅读性进行格式化后的代码

// oldCh 是一个旧虚拟节点数组
if (isUndef(oldKeyToIdx)) {
    oldKeyToIdx = createKeyToOldIdx(oldCh, oldStartIdx, oldEndIdx)
}
if (isDef(newStartVnode.key)) {
    // map 方式获取
    idxInOld = oldKeyToIdx[newStartVnode.key]
} else {
    // 遍历方式获取
    idxInOld = findIdxInOld(newStartVnode, oldCh, oldStartIdx, oldEndIdx)
}
```
创建map函数
```javascript
function createKeyToOldIdx(children, beginIdx, endIdx) {
    let i, key
    const map = {}
    for (i = beginIdx; i <= endIdx; ++i) {
        key = children[i].key
        if (isDef(key)) map[key] = i
    }
    return map
}
```
遍历寻找
```javascript
// sameVnode 是对比新旧节点是否相同的函数
function findIdxInOld(node, oldCh, start, end) {
    for (let i = start; i < end; i++) {
        const c = oldCh[i]

        if (isDef(c) && sameVnode(node, c)) return i
    }
}
```

## Vue基本代码结构
```javascript
const vm = new Vue({
    el: '#app', //所有的挂载元素会被 Vue 生成的 DOM 替换
    data: {}, //this -> window
    methods: {}, //this -> vm
    //注意，不应该使用箭头函数来定义 method 函数 ,this将不再指向vm实例
    props: {}, // 可以是数组或对象类型，用于接收来自父组件的数据
    //对象允许配置高级选项，如类型检测、自定义验证和设置默认值
    watch: {}, //this -> vm
    computed: {},
    render() {},
    // 声明周期钩子函数
})
```
当一个Vue实例被创建时，它将data对象中的所有的property加入到Vue的响应式系统中。当这些property的值发生改变时，视图将会产生 响应，即匹配更新为新的值。
例外：
- Vue实例外部新增的属性改变时不会更新视图。
- Object.freeze()，会阻止修改现有的property，响应系统无法追踪其变化。

实例属性和方法：
- 访问el属性：vm.$el,`document.getElemnetById(‘app’)``;
- 访问data属性：vm.$data
- 以_或$开头的property不会被Vue实例代理，因为它们可能和Vue内置的property、API方法冲突。你可以使用例如vm.$data._property的方式访问这些property。
- data._property的方式访问这些property。
- 访问data中定义的变量：vm.a,vm.$data.a
- 访问methods中的方法：vm.方法名()
- 访问watch方法：vm.$watch()

不要在选项property或回调上使用箭头函数,this将不会指向Vue实例 比如created: () => console.log(this.a)或vm.$watch('a', newValue => this.myMethod())。
因为箭头函数并没有this，this会作为变量一直向上级词法作用域查找，直至找到为止，经常导致Uncaught TypeError: Cannot read property of undefined或Uncaught TypeError: this.myMethod is not a function之类的错误。

## Vue指令
**插入数据：**
- 插值表达式相当于占位符，不会清空元素中的其他内容。直接写在标签中。会将html标签作为文本显示。
- v-text会覆盖元素中原本的内容。写在开始标签中，以属性的形式存在。会将html标签作为文本显示。
- v-html（innerHTML）会覆盖元素中原本的内容，会将数据解析成html标签。

## Vue组件
**组件配置对象和vue实例的区别**
- 组件配置对象没有el，组件模板定义在template中；
- 组件配置对象中data是函数，该函数返回的对象作为数据。

**创建组件模板**
方法一
```javascript
var com = Vue.extend({
    //通过template属性 指定组件要展示的html结构
    template: '<h3>这是使用Vue.extend搭建的全局组件</h3>'
})
```
方法二：使用对象创建模板
```javascript
{
    template: '<h3>这是使用Vue.extend搭建的全局组件-com3</h3>'
}
```
方法三
```html
<template id="tmpl"> 写在受控区域外面
    ......
</template> 

{ template:'#tmpl'  }
```
**组件中的data是一个函数的原因**
- 多次使用该组件，如果修改其中一个中的数据，另一个也会改变。
- 写成函数的形式，每次调用函数，返回一个新的对象

**ref属性**
- 获取dom元素/组件：标签上添加ref属性，this.$refs.ref属性值获取该dom元素/组件。
- this.$refs.ref属性值.变量名获取组件中的数据
- this.$refs.ref属性值.方法名()获取组件中的方法
- $parent 和 $children 获取 父/子组件的数据和方法
- this.$parent获取父组件
- $children由于子组件的个数不确定 返回的是一个数组 ,不是对象
- this.$children[0]获取第一个子组件

作用域插槽：父组件替换插槽的标签，内容由子组件决定。
编译的作用域：自身的数据在自身模板template标签中生效
插槽上添加 属性绑定：data=’子组件中的数据’
父组件通过template标签，添加slot-scope=’slot’ slot-scope属性接收子组件中的数据slot.data
template标签中的html结构替换slot插槽中的默认html结构。

## Vue和Jquery相比的优缺点
Vue适用的场景：复杂数据操作的后台页面，表单填写及验证页面；
Vue侧重数据绑定；（框架）
jQuery适用的场景：html5的动画页面，需要js来操作页面样式的页面；
jQuery侧重样式操作以及动画效果等。（类库）

## Vue和React的区别
1. 监听数据变化的实现原理不同：
    Vue通过getter/setter以及数据劫持来实现监听数据的变化 React则是通过比较引用（diff算法）来实现，如果不优化可能导致大量不必要的VDOM的重新渲染。 两者设计理念不同，Vue使用的是可变的数据，React则强调数据的不可变，Vue更加简单，React构建大型应用的时候更加鲁棒。
2. 数据流向不同
    Vue2.0中组件与DOM可以通过v-model双向绑定，2.0中不再支持父子组件之间通过props双向绑定，不鼓励组件对props进行修改，会警告。 React提倡单向数据流，使用Redux进行数据流状态管理
3. 组件间的通信方式不同
    Vue的组件通信由三种方式：父组件通过props向子组件传递数据或者回到，子组件通过事件想父组件传递消息，通过2.2.0中新增的provide/inject实现父组件向子组件注入数据，可以跨越多个层级。 React的组件通信有三种方式：父组件通过props可以向子组件传递数据或者回调，可以通过context进行跨层级的通信，这其实和provide/inject起到的作用差不多，React本身不支持自定义时间，React中子组件向父组件传递消息使用回调函数。
4. 模板渲染方式不同
    React通过JSX渲染模板，Vue通过拓展的HTML语法进行渲染。 React是在组件的js代码中，通过原生js来实现模板中的常见语法，比如插值，条件，循环等等，更加纯粹原生，而Vue是在和组件js代码分离的单独的模板中通过指令来实现的。
5. 渲染过程不同
    Vue可以更快计算出虚拟DOM的差异，因为渲染过程中，Vue会收集依赖，然后根据注册的组件渲染对应的DOM，不需要重新渲染每一个组件 React在应用的状态改变时，全部子组件都会重新渲染。
6. 框架本质不同
    Vue本质是MVVM框架，由MVC发展而来； React是前端组件化框架，由后端组件化发展而来。
7. Vuex和Redux的区别
    Redux使用的是不可变数据，而Vuex的数据是可变的，因此，Redux每次都是用新state替换旧state，而Vuex是直接修改。Redux在检测数据变化的时候，是通过diff的方式比较差异的，而Vuex其实和Vue的原理一样，是通过getter/setter来比较的，这两点的区别，也是因为React和Vue的设计理念不同。React更偏向于构建稳定大型的应用，非常的科班化。相比之下，Vue更偏向于简单迅速的解决问题，更灵活，不那么严格遵循条条框框。

## 组件间的通信
父组件向子组件传值总结：
1. 子组件在props中创建一个属性，用以接收父组件传过来的值
2. 父组件中注册子组件
3. 在子组件标签中添加子组件props中创建的属性
4. 把需要传给子组件的值赋给该属性

子组件向父组件传值总结：
1. 子组件中需要以某种方式例如点击事件的方法来触发一个自定义事件
2. 将需要传的值作为$emit的第二个参数，该值将作为实参传给响应自定义事件的方法
3. 在父组件中注册子组件并在子组件标签上绑定对自定义事件的监听

在通信中，无论是子组件向父组件传值还是父组件向子组件传值，他们都有一个共同点就是有中间介质，子向父的介质是自定义事件，父向子的介质是props中的属性。抓准这两点对于父子通信就好理解了

vue兄弟组件之间的传值：
1. 创建一个事件总线，例如demo中的eventBus.js，new一个vue对象并向外暴露，用它作为通信桥梁 2.在需要传值的组件中引入eventBus.js，用bus.$emit触发一个自定义事件，并传递参数
2. 在需要接收数据的组件中引入eventBus.js，在mounted() 钩子函数(挂载实例) 中用bus.$on监听自定义事件，并在回调函数中处理传递过来的参数

## Vue常用指令
- v-once：能执行一次性地插值，当数据改变时，插值处的内容不会更新。
- v-show：和v-if一样，区别是if是将代码注释掉，v-show是给一个display：none的属性 让它不显示。v-if 有更高的切换消耗而 v-show 有更高的初始渲染消耗。因此，如果需要频繁切换使用 v-show 较好，如果在运行时条件不大可能改变则使用 v-if 较好。
- v-for：基于数据渲染一个列表，类似于JS中的遍历。其数据类型可以是 Array | Object | number | string。
- v-html：双大括号会将数据解释为普通文本，而非HTML 代码。为了输出真正的 HTML，你需要使用 v-html 而且给一个标签加了v-html 里面包含的标签都会被覆盖。
- v-text：给一个便签加了v-text 会覆盖标签内部原先的内容
- v-bind：动态属性绑定，缩写:
- v-on：动态事件绑定，缩写@
- v-model：双向数据绑定，限制在&lt;input&gt;、&lt;select&gt;、&lt;textarea&gt;、components中使用

## 一个指令定义对象可以提供如下几个钩子函数 (均为可选)：
- inserted：被绑定元素插入父节点时调用 (仅保证父节点存在，但不一定已被插入文档中)。
- bind：只调用一次，指令第一次绑定到元素时调用。在这里可以进行一次性的初始化设置。
- update：所在组件的 VNode 更新时调用，但是可能发生在其子 VNode 更新之前。指令的值可能发生了改变，也可能没有。但是你可以通过比较更新前后的值来忽略不必要的模板更新 (详细的钩子函数参数见下)。
- componentUpdated：指令所在组件的 VNode 及其子 VNode 全部更新后调用。
- unbind：只调用一次，指令与元素解绑时调用。


## 指令钩子函数会被传入以下参数
- el：指令所绑定的元素，可以用来直接操作 DOM 。
- binding：一个对象，包含以下属性：
- name：指令名，不包括 v- 前缀。
- value：指令的绑定值，例如：v-my-directive=“1 + 1” 中，绑定值为 2。
- oldValue：指令绑定的前一个值，仅在 update 和 componentUpdated 钩子中可用。无论值是否改变都可用。
- expression：字符串形式的指令表达式。例如 v-my-directive=“1 + 1” 中，表达式为 “1 + 1”。
- arg：传给指令的参数，可选。例如 v-my-directive:foo 中，参数为 “foo”。
- modifiers：一个包含修饰符的对象。例如：v-my-directive.foo.bar 中，修饰符对象为 { foo: true, bar: true }。
- vnode： Vue编译生成的虚拟节点。移步 VNode API 来了解更多详情。
- oldVnode：上一个虚拟节点，仅在 update 和 componentUpdated 钩子中可用。

## Vue的生命周期
一个组件从开始到最后消亡所经历的各种状态，就是一个组件的生命周期：
- beforeCreate()
    说明：在实例初始化之后，数据观测 (data observer) 和 event/watcher 事件配置之前被调用
    注意：此时，无法获取 data中的数据、methods中的方法
- created()
    说明：这是一个常用的生命周期，可以调用methods中的方法、改变data中的数据
- beforeMounted()
    说明：在挂载开始之前被调用,此时无法获取到el中的DOM元素
- mounted()
    说明：此时，vue实例已经挂载到页面中，可以获取到el中的DOM元素，进行DOM操作
- beforeUpdated()
    说明：数据更新时调用，发生在虚拟 DOM 重新渲染和打补丁之前。你可以在这个钩子中进一步地更改状态，这不会触发附加的重渲染过程。
    注意：此处获取的数据是更新后的数据，但是获取页面中的DOM元素是更新之前的
- updated()
    说明：组件 DOM 已经更新，所以你现在可以执行依赖于 DOM 的操作
- beforeDestroy()
    说明：实例销毁之前调用。在这一步，实例仍然完全可用。
    使用场景：实例销毁之前，执行清理任务，比如：清除定时器等
- destroyed()
    说明：Vue 实例销毁后调用。调用后，Vue 实例指示的所有东西都会解绑定，所有的事件监听器会被移除，所有的子实例也会被销毁。

## Vue父子组件生命周期执行顺序
- 加载渲染过程
    父beforeCreate->父created->父beforeMount->子beforeCreate->子created->子beforeMount->子mounted->父mounted
- 子组件更新过程
    父beforeUpdate->子beforeUpdate->子updated->父updated
- 父组件更新过程
    父beforeUpdate->父updated
- 销毁过程
    父beforeDestroy->子beforeDestroy->子destroyed->父destroyed

## Vuex是什么？怎么使用？那种功能场景使用它
只要来读取的状态集中放在store中；改变状态的方式就是提交mutations。这是个同步的实物；异步逻辑应该封装中action中。
- state:Vuex 使用单一状态树，既每个应用将仅仅包含一个store实例，单单一状态树和模块化并不冲突。存放的数据状态，不可以直接修改里面的数据。可以通过this.$store.state调用state中的参数。
- mutations:mutations定义的方法动态修改Vuex的store中的状态或数据。Mutatison中必须是同步函数。可以通过this.$store.commit('increment',args)来调用并更改state中的参数状态
- getters:类似vue的计算属性，主要用来过滤一些数据。
- action:action可以理解为通过mutations里面处理数据的方法变成可异步的处理数据的方法，简单的说就是异步操作数据。通过this.$store.dispatch('increment')来调用action方法。Action提交的是mutation，而不是直接改变装填，action可以包含异步操作 Modeules:项目特别复杂的时候，可以让每一个模块拥有自己的state，mutation，action，getters，使得结构非常清晰，方便管理

## Vuex中的数据流向：
1. vue组件先调用dispatch 来触发Actions做些异步处理或批量的同步操作，紧接着Actions通过提交commit来调用Mutations , Mutations 中放的是一个一个同步的对state的修改,只有通过mutations才能改变公用数据的值。最后通过this.$store.state.id的方法将值映射到页面上。
2. 如果逻辑简单，vue 组件也可以略过actions, 让组件直接调用mutations来修改state的公用数据的值

## vue-router的概念与使用：
vue-router路由管理器
vue router和vue.js的核心深度集成，可以方便的用于spa的应用程序开发
它的功能有：
- 支持HTML5 history模式，和hash模式；支持嵌套路由；支持路由参数，支持编程式路由，支持命名路由。
- 路由的进阶，导航守卫，路由元信息，过渡效果，数据获取，滚动行为，路由懒加载。

## vue-router的基本使用
基本使用步骤：
1. 引入相关的库文件
    ```html
    <script src="./lib/vue-routerxxx.js"></script>
    ```
2. 添加路由连接
    ```html
    <router-link to="/user">User</router-link>
    ```
3. 添加路由填充位
    ```html
    <router-view></router-view>
    ```
4. 定义路由组
    ```javascript
    var User = {template: '<div>user</div>'}
    ```
5. 配置路由规则并创建路由实例，

## 后记
这些问题我都会写文章补上，也欢迎大家给我提供我这里没有的问题哦~