### 逻辑层与物理层

早期开发页面的时候就是基于浏览器的 dom api 操作 dom 来做渲染和交互，但是 dom api 比较啰嗦，而且当时浏览器的兼容性问题也比较麻烦，不同浏览器有不同的写法。为了简化 dom 操作和更方便的兼容各种浏览器，出现了 jquery 并且迅速流行开来。

1、dom 就算是物理层，jquery 是操作 dom 的一系列工具函数，也是工作在物理层。

2、网页做的事情基本就是拿到数据渲染 dom，并且数据改变之后更新 dom，这个流程是通用的，后来逐渐出现了 mvvm 框架，来自动把数据的变更映射到 dom，不再需要手动操作 dom。也就是 vue、react 等现代的前端框架。我把这一层叫做逻辑层。

### 现代前端框架

前端框架是 UI = f(state) 这种声明式的思想，只需要声明组件的视图、组件的状态数据、组件之间的依赖关系，那么状态改变就会自动的更新 dom。而 jquery 那种直接操作 dom 的工具函数库则是命令式的。

#### 视图描述

1、react

react 是给 js 扩展了 jsx 的语法，由 babel 实现，可以在描述视图的时候直接用 js 来写逻辑。

2、vue

vue 是实现了一套 template，引入了插值、指令、过滤器等模版语法，相对于 jsx 来说更简洁，template 的编译器由 vue 实现。

3、优缺点

vue template 是受限制的，只能访问 data，prop、method，可以静态的分析和优化，而 react 的 jsx 因为直接是 js 的语法，动态逻辑比较多，没法静态的做分析和优化。

但是 vue template 也不全是好处，因为和 js 上下文割裂开来，引入 typescript 做类型推导的时候就比较困难，需要单独把所有 prop、method、data 的类型声明一遍才行。而 react 的 jsx 本来就是和 js 同一个上下文，结合 typescript 就很自然。

#### 数据驱动

1、数据变化的检测基本只有三种方式：watch、脏检查和不检查。

vue 就是基于数据的 watch 的，组件级别通过 Object.defineProperty 监听对象属性的变化，重写数组 api 监听数组元素的变化，之后进行 dom 的更新。

angular 则是基于脏检查，在每个可能改变数据的逻辑之后都对比下数据是否变了，变了的话就去更新 dom。

react 则是不检查，不检查是因为不直接渲染到 dom，而是中间加了一层虚拟 dom，每次都渲染这个虚拟 dom，然后 diff 下渲染的虚拟 dom 是否变了，变了的话就去更新对应的 dom。

2、**vue 和 react 优化**

vue 是组件级别的数据 watch，当组件内部监听数据变化的地方特别多的时候，一次更新可能计算量特别大，计算量大了就可能会导致丢帧，也就是渲染的卡顿。所以 vue 的优化方式就是把大组件拆成小组件，这样每个数据就不会有太多的 watcher 了。

react 并不监听数据的变化，而是渲染出整个虚拟 dom，然后 diff。基于这种方案的优化方式就是对于不需要重新生成 vdom 的组件，通过 shouldComponentUpdate 来跳过渲染。

但是当应用的组件树特别大的时候，只是 shouldComponentUpdate 跳过部分组件渲染，依然可能计算量特别大。计算量大了同样可能导致渲染的卡顿，怎么办呢？

树的遍历有深度优先和广度优先两种方式，组件树的渲染就是深度优先的，一般是通过递归来做，但是如果能通过链表记录下路径，就可以变成循环。变成了循环，那么就可以按照时间片分段，让 vdom 的生成不再阻塞页面渲染，这就像操作系统对多个进程的分时调度一样。

react fiber：通过组件树改成链表，把 vdom 的生成从递归改循环的功能。

react 不监听、不检查数据变化，每次都渲染生成 vdom，然后进行 vdom 的对比，那么优化的思路就是 shouldComponentUpdate 来跳过部分组件的 render，而且 react 内部也做了组件树的链表化（fiber）来把递归改成可打断的渲染，按照时间片来逐渐生成整个 vdom。

#### 组件复用

vue 的组件是 option 对象的方式，那么逻辑复用方式很自然可以想到通过对象属性的 mixin，vue2 的组件内逻辑复用方案就是 mixin，但是 mixin 很难区分混入的属性、方法的来源，比较乱，代码维护性差。但也没有更好的方案。

```js
export const myMixin = {
  data() {
    return {
      num: 1,
    };
  },
  created() {
    this.hello();
  },
  methods: {
    hello() {
      console.log("hello from mixin");
    },
  },
};

// 混入组件
export default {
  mixins: [myMixin],
};
```

react 的组件是 class 和 function 两种形式，那么类似高阶函数的高阶组件（high order component）的方式就比较自然，也就是组件套组件，在父组件里面执行一部分逻辑，然后渲染子组件。

除了多加一层组件的 HOC 方式以外，没有逻辑的部分可以直接把那部分 jsx 作为 props 传入另一个组件来复用，也就是 render props。

HOC 和 render props 是 react 的 class 组件支持的两种逻辑复用方案。但是 HOC 的逻辑复用方式最终导致了组件嵌套太深，而且 class 内部生命周期比较多，逻辑都放在一起导致了组件比较大。

**解决 react 组件复用缺点**

react 就在 function 组件的 fiber 节点中加入了 memorizedState 属性用来存储数据，然后在 function 组件里面通过 api 来使用这些数据，这些 api 被叫做 hooks api。

#### hooks api

因为是使用 fiber 节点上的数据，就把 api 命名为了 useXxx。

每个 hooks api 都要有自己存放数据的地方，怎么组织？

hooks 使用了数组的方式。当然，实现起来用的是链表。

用数组的话顺序不能变，所以 hooks api 不能出现在 if 等逻辑块中，只能在顶层。

每个 hooks api 取对应的 fiber.memoriedState 中的数据来用。

hook api 可以分为 3 类

1、数据类

- useState： 在 fiber.memoriedState 的对应元素中存放数据
- useMemo：在 fiber.memoriedState 的对应元素中存放数据，值是缓存的函数计算的结果，在 state 变化后重新计算值
- useCallback：在 fiber.memoriedState 的对应元素中存放数据，值是函数，在 state 变化后重新执行函数，是 useMemo 在值为函数的场景下的简化 api，比如 useCallback(fn, [a,b]) 相当于 useMemo(() => fn, [a, b])
- useReducer：在 fiber.memoriedState 的对应元素中存放数据，值为 reducer 返回的结果，可以通过 action 来触发值的变更。
- useRef：在 fiber.memoriedState 的对应元素中存放数据，值为 {current: 具体值} 的形式，因为对象不变，只是 current 属性变了，所以不会修改。

useState 是存储值最简单的方式，useMemo 是基于 state 执行函数并且缓存结果，相当于 vue 的 getter，useCallback 是一种针对值为函数的情况的简化，useReducer 是通过 action 来触发值的修改。useRef 包了一层对象，每次对比都是同一个，所以可以放一些不变的数据。

2、逻辑类

- useEffect：异步执行函数，当依赖 state 变化之后会再次执行，当组件销毁的时候会调用返回的清理函数。
- useLayoutEffect：在渲染完成后同步执行函数，可以拿到 dom

3、ref 转发

数据可以通过各种方案共享，但是 dom 元素这种就得通过 ref 转发了，所谓的 ref 转发就是在父组件创建 ref，然后子组件把元素传过去。传过去之前想做一些修改，就可以用 useImperativeHandle 来改。

4、解决复用的问题

逻辑扩展不需要嵌套 hoc 了，多调用一个自定义的 hooks 就行

组件的逻辑也不用都写在 class 里了，完全可以抽离成不同的 hooks

react 通过 function 组件的 hooks api 解决了 class 组件的逻辑复用方案的问题。（fiber 是解决性能问题的，而 hooks 是解决逻辑复用问题的）

<!-- todo: hoc与hooks 区别 -->
