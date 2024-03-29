### vdom 和 fiber

React 16 之后引入了 fiber 架构，就是在这里做了改变，它不是直接渲染 vdom 了，而是先转成 fiber。

本来 vdom 里通过 children 关联父子节点，而 fiber 里面则是通过 child 关联第一个子节点，然后通过 sibling 串联起下一个，所有的节点可以 return 到父节点。

这样把一颗 vdom 树，变成了 fiber 链表。

**fiber 架构的意义**

之前是递归渲染 vdom 的，然后 diff 下来做 patch 的渲染，渲染和 diff 时递归进行的。

先把 vdom 转 fiber。也就是 reconcile 的过程。 因为 fiber 是链表，就可以打断，用 schedule 来空闲时调度(requestIdleCallback)就行，最后全部转完之后，再一次性 render，这个过程叫做 commit。

这样，之前只有 vdom 的 render 和 patch，现在却变成了 vdom 转 fiber 的 reconcile，空闲调度 reconcile 的 schedule，最后把 fiber 渲染的 commit 三个阶段。

**意义就在于可打断上。因为递归渲染 vdom 可能耗时很多，js 计算量大了会阻塞渲染，而 fiber 是可打断的，就不会阻塞渲染，而且还会在这个过程中把需要用到的 dom 创建好，做好 diff 来确定是增是删还是改。**

### schedule

**schedule 就是通过空闲调度每个 fiber 节点的 reconcile（vdom 转 fiber），全部 reconcile 完了就执行 commit。**

```js
let nextFiberReconcileWork = null;
let wipRoot = null;

function workLoop(deadline) {
  let shouldYield = false;
  while (nextFiberReconcileWork && !shouldYield) {
    nextFiberReconcileWork = performNextWork(nextFiberReconcileWork);
    shouldYield = deadline.timeRemaining() < 1;
  }

  if (!nextFiberReconcileWork) {
    commitRoot();
  }

  requestIdleCallback(workLoop);
}

function performNextWork(fiber) {
  reconcile(fiber);

  if (fiber.child) {
    return fiber.child;
  }

  let nextFiber = fiber;

  while (nextFiber) {
    if (nextFiber.sibling) {
      return nextFiber.sibling;
    }
    nextFiber = nextFiber.return;
  }
}

requestIdleCallback(workLoop);
```

### reconcile

**reconcile 负责 vdom 转 fiber，并且还会准备好要用的 dom 节点、确定好是增、删、还是改，通过 schdule 的调度，最终把整个 vdom 树转成了 fiber 链表。**

reconcile 是 vdom 转 fiber，但还会做两件事：一个是提前创建对应的 dom 节点，一个是做 diff，确定是增、删还是改。

```js
function render(element, container) {
  wipRoot = {
    dom: container,
    props: {
      children: [element],
    },
  };
  nextFiberReconcileWork = wipRoot;
}

function reconcile(fiber) {
  if (!fiber.dom) {
    fiber.dom = createDom(fiber);
  }

  // fiber.props.children 就是vdom的节点
  // reconcileChildren：vdom转child、sibling、return这样串联起来的fiber链表
  reconcileChildren(fiber, fiber.props.children);
}

function reconcileChildren(wipFiber, elements) {
  let index = 0;
  let prevSibling = null;

  // 循环处理每一个vdom的elements, 如果index == 0，那就是串联，否则是sibling串联。

  while (index < elements.length) {
    const element = elements[index];
    let newFiber = {
      type: element.type,
      props: element.props,
      dom: null,
      return: wipFiber,
      effectTag: "PLACEMENT",
    };

    if (index === 0) {
      wipFiber.child = newFiber;
    } else if (element) {
      prevSibling.sibling = newFiber;
    }

    prevSibling = newFiber;
    index++;
  }
}
```

### commit

**commit 负责把 reconcile 产生的 fiber 链表一次性添加到 dom 中，因为 fiber 对应的节点提前创建好了、是增是删还是改也都知道了，所以，这一个阶段很快。**

```js
if (!nextFiberReconcileWork) {
  commitRoot();
}

// 从根 fiber 开始 commit，并且把 wipRoot 设置为空。
function commitRoot() {
  commitWork(wipRoot.child);
  wipRoot = null;
}

//  fiber 节点的渲染就是按照 child、sibling 的顺序以此插入到 dom 中
function commitWork(fiber) {
  if (!fiber) {
    return;
  }

  let domParentFiber = fiber.return;
  while (!domParentFiber.dom) {
    domParentFiber = domParentFiber.return;
  }
  const domParent = domParentFiber.dom;

  if (fiber.effectTag === "PLACEMENT" && fiber.dom != null) {
    domParent.appendChild(fiber.dom); // fiber节点都要往上找它的父节点，只是新增，只需要appendChild
  }

  commitWork(fiber.child);
  commitWork(fiber.sibling);
}
```

dom 已经在 reconcile 节点就创建好了。

```js
function createDom(fiber) {
  const dom =
    fiber.type == "TEXT_ELEMENT"
      ? document.createTextNode("")
      : document.createElement(fiber.type);

  for (const prop in fiber.props) {
    setAttribute(dom, prop, fiber.props[prop]);
  }

  return dom;
}

function isEventListenerAttr(key, value) {
  return typeof value == "function" && key.startsWith("on");
}

function isStyleAttr(key, value) {
  return key == "style" && typeof value == "object";
}

function isPlainAttr(key, value) {
  return typeof value != "object" && typeof value != "function";
}

const setAttribute = (dom, key, value) => {
  if (key === "children") {
    return;
  }

  if (key === "nodeValue") {
    dom.textContent = value;
  } else if (isEventListenerAttr(key, value)) {
    const eventType = key.slice(2).toLowerCase();
    dom.addEventListener(eventType, value);
  } else if (isStyleAttr(key, value)) {
    Object.assign(dom.style, value);
  } else if (isPlainAttr(key, value)) {
    dom.setAttribute(key, value);
  }
};
```

这在 reconcile 时就做好了，commit 自然很快。

### 总结

1、fiber 打断原理

把 vdom 转化为链表结构（fiber），render 阶段会找到 vdom 中变化的部分，创建 dom，打上增删改的标记，这个叫做 reconcile，调和。链表结构记录了兄弟节点和父节点信息。

把 vdom 转换成 fiber，从原先的递归换成了循环，这样就可以打断了。

循环处理前判断待处理的任务队列有没有优先级更高的任务，有的话就先处理那边的 fiber，这边的先暂停一下。

2、commit

commit 阶段就根据标记来更新 dom 就可以了。

但是 commit 阶段要再遍历一次 fiber 来查找有 effectTag 的节点，更新 dom 么？

完全可以在 reconcile 的时候把有 effectTag 的节点收集到一个队列里，然后 commit 阶段直接遍历这个队列就行了。

这个队列叫做 effectList。

react 会在 commit 阶段遍历 effectList，根据 effectTag 来增删改 dom。

dom 创建前后就是 useEffect、useLayoutEffect 还有一些函数组件的生命周期函数执行的时候。

useEffect 被设计成了在 dom 操作前异步调用，useLayoutEffect 是在 dom 操作后同步调用。

因为都要操作 dom 了，这时候如果来了个 effect 同步执行，计算量很大，那不是把 fiber 架构带来的优势有毁了么？

所以 effect 是异步的，不会阻塞渲染。

而 useLayoutEffect，顾名思义是想在这个阶段拿到一些布局信息的，dom 操作完以后就可以了，而且都渲染完了，自然也就可以同步调用了。

实际上 react 把 commit 阶段也分成了 3 个小阶段。

before mutation、mutation、layout。

mutation 就是遍历 effectList 来更新 dom 的。

它的之前就是 before mutation，会异步调度 useEffect 的回调函数。

它之后就是 layout 阶段了，因为这个阶段已经可以拿到布局信息了，会同步调用 useLayoutEffect 的回调函数。而且这个阶段可以拿到新的 dom 节点，还会更新下 ref。
