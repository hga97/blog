### React、JSX 和 DOM 元素之间的关系

```jsx
const element = <h1 title="foo">Hello</h1>;
const container = document.getElementById("root");
ReactDOM.render(element, container);
```

#### JSX 通过 Babel 转换为 JS

调用 createElement 替换标签中的代码，传递标签名称、props 和 children 作为参数。

- type: 创建的 DOM 节点类型的字符串
- props: JSX 属性的所有键值对，特殊的属性 children
- children: 通常是一个具有更多元素的数组

```jsx
const element = <h1 title="foo">Hello</h1>;

// 转换为

const element = React.createElement("h1", { title: "foo" }, "helllo");

// 转换为

const element = {
  type: "h1",
  props: {
    title: "foo",
    children: "Hello",
  },
};
```

#### 替换 ReactDOM.render

```jsx
ReactDOM.render(element, container);

// 转化为

const node = document.createElement(element.type);
node["title"] = element.props.title;

const text = document.createTextNode("");
text["nodeValue"] = element.props.children;

node.appendChild(text);
container.appendChild(node);
```

### createElement 函数

```jsx
const element = (
  <div id="foo">
    <a>bar</a>
    <b />
  </div>
);

const container = document.getElementById("root");
ReactDOM.render(element, container);
```

1、JSX 转换 JS

```jsx
/** @jsx Didact.createElement */
const element = (
  <div id="foo">
    <a>bar</a>
    <b />
  </div>
);

// 转换为

const element = Didact.createElement(
  "div",
  { id: "foo" },
  Didact.createElement("a", null, "bar"),
  Didact.createElement("b")
);
```

2、编写 createElement 函数

```jsx
const Didact = {
  createElement,
};

function createTextElement(text) {
  return {
    type: "TEXT_ELEMENT",
    props: {
      nodeValue: text,
      children: [],
    },
  };
}

function createElement(type, props, ...children) {
  return {
    type,
    props: {
      ...props,
      children: children.map((child) =>
        typeof child === "object" ? child : createTextElement(child)
      ),
    },
  };
}
```

### render 函数

```jsx
const Didact = {
  render,
};

function render(element, container) {
  const dom =
    element.type == "TEXT_ELEMENT"
      ? document.createTextNode("")
      : document.createElement(element.type);

  const isProperty = (key) => key !== "children";

  Object.keys(element.props)
    .filter(isProperty)
    .forEach((name) => (dom[name] = element.props[name]));

  element.props.children.forEach((child) => render(child, dom));

  container.appendChild(dom);
}

const container = document.getElementById("root");
Didact.render(element, container);
```

### 并发模式

1、递归调用存在的问题：

**将 vdom 转换为 dom 之前，不会停止。如果元素树很大，可能会阻止主线程太久。
浏览器需要做高优先级的事件，如用户输入或者保持动画流畅，则必须等待渲染完成。**

2、解决

**把工作分成小单元，在完成每个单元后，如果还有其他事情要做，我们会让浏览器中断渲染。**

使用 requestIdleCallback 来制作一个循环，将 requestIdleCallback 视为 setTimeout，但我们不会告诉它何时运行，而是在主线程空闲时，浏览器将运行回调。

?> requestIdleCallback  
这个函数将在浏览器空闲时期被调用。这使开发者能够在主事件循环上执行后台和低优先级工作，而不会影响延迟关键事件，如动画和输入响应。函数一般会按先进先调用的顺序执行，然而，如果回调函数指定了执行超时时间 timeout，则有可能为了在超时前执行函数而打乱执行顺序。

?> IdleDeadline  
做为一个 IdleDeadline interface 类型的参数传递给 requestIdleCallback。它提供了一个方法 timeRemaining()，可以让你判断用户代理 (浏览器) 还剩余多少闲置时间可以用来执行耗时任务。

```jsx
let nextUnitOfWork = null
​
function workLoop(deadline) {
  let shouldYield = false
  while (nextUnitOfWork && !shouldYield) {
    nextUnitOfWork = performUnitOfWork(
      nextUnitOfWork
    )
    shouldYield = deadline.timeRemaining() < 1
  }
  requestIdleCallback(workLoop)
}
​
requestIdleCallback(workLoop)
​
// 该函数不仅执行工作，还返回下一个工作单元。
function performUnitOfWork(nextUnitOfWork) {
  // TODO
}
```

### fiber

#### 数据结构

```js
Didact.render(
  <div>
    <h1>
      <p />
      <a />
    </h1>
    <h2 />
  </div>,
  container
);
```

![fiber](https://img-blog.csdnimg.cn/f530237d7aed487fb2dbbc6fee51e06f.png)

这个数据结构的目标之一是让找到下一个工作单位变得容易。每个 fiber 都与其第一个子节点、下一个兄弟和父节点有关系。

例子：

1、fiber 上完成工作时，如果它有 child，该 fiber 将是下一个工作单元。  
（div fiber 工作时，下一个工作单元时 h1 fiber）

2、如果 fiber 没有 child，使用 sibling 作为下一个工作单位。  
（p fiber 没有 child，所以在完成后移动到 a fiber）

3、如果 fiber 没有 child，也没有 sibling，就去找 parent 的 sibling。  
（a 和 h2 fiber）

4、如果 parent 没有 sibling，继续通过父节点，直到我们找到一个有 sibling 的，或者直到根节点。

5、直到根节点我们的 render 执行完毕

#### performUnitOfWork

1、在 render 中，我们将创建根 fiber，并将其设置为 nextUnitOfWork。

2、其余的工作将在 performUnitOfWork 函数上进行，为每个 fiber 做三件事。

- 将元素添加到 dom 中
- 为元素的子元素创建 fiber
- 选择下一个工作单元

```js
function createDom(fiber) {
  const dom =
    fiber.type == "TEXT_ELEMENT"
      ? document.createTextNode("")
      : document.createElement(fiber.type)
​
  const isProperty = key => key !== "children"
  Object.keys(fiber.props)
    .filter(isProperty)
    .forEach(name => {
      dom[name] = fiber.props[name]
    })
​
  return dom
}

function render(element, container) {
  nextUnitOfWork = {
    dom: container,
    props: {
      children: [element],
    },
  }
}
​
let nextUnitOfWork = null

function workLoop(deadline) {
  let shouldYield = false
  while (nextUnitOfWork && !shouldYield) {
    nextUnitOfWork = performUnitOfWork(
      nextUnitOfWork
    )
    shouldYield = deadline.timeRemaining() < 1
  }
  requestIdleCallback(workLoop)
}
​
requestIdleCallback(workLoop)


function performUnitOfWork(fiber) {
  // TODO add dom node
  if (!fiber.dom) {
    fiber.dom = createDom(fiber)
  }
​
  if (fiber.parent) {
    fiber.parent.dom.appendChild(fiber.dom)
  }
​
  // TODO create new fibers
   const elements = fiber.props.children
  let index = 0
  let prevSibling = null
​
  while (index < elements.length) {
    const element = elements[index]
​
    const newFiber = {
      type: element.type,
      props: element.props,
      parent: fiber,
      dom: null,
    }

    if (index === 0) {
      fiber.child = newFiber
    } else {
      prevSibling.sibling = newFiber
    }
​
    prevSibling = newFiber
    index++
  }

  // TODO return next unit of work
 if (fiber.child) {
    return fiber.child
  }

  let nextFiber = fiber
  while (nextFiber) {
    if (nextFiber.sibling) {
      return nextFiber.sibling
    }
    nextFiber = nextFiber.parent
  }

}

```

### 渲染和提交阶段
