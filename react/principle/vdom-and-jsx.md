### vdom 的渲染

1、vdom

vdom 全称 virtual dom，用来声明式的描述页面，现代前端框架很多都基于 vdom。前端框架负责把 vdom 转为对真实 dom 的增删改，也就是 vdom 的渲染。

dom 主要是元素、属性、文本，vdom 也是一样，其中元素是{type,props,children}的结构，文本就是字符串、数字。

一段 vdom：

```js
{
    type: 'ul',
    props: {
        className: 'list'
    },
    children: [
        {
            type: 'li',
            props: {
                className: 'item',
                style: {
                    background: 'blue',
                    color: '#fff'
                },
                onClick: function() {
                    alert(1);
                }
            },
            children: [
                'aaaa'
            ]
        },
        {
            type: 'li',
            props: {
                className: 'item'
            },
            children: [
                'bbbbddd'
            ]
        },
        {
            type: 'li',
            props: {
                className: 'item'
            },
            children: [
                'cccc'
            ]
        }
    ]
}
```

它描述的是一个 ul 的元素，它有三个 li 子元素，其中第一个子元素有 style 的样式和 onclick 的事件。

前端框架就是通过这样的对象结构来描述界面的，然后把它渲染到 dom。

2、vdom 如何渲染

需要递归，对不同的类型做不同的处理。

- 文本类型，用 document.createTextNode 来创建文本节点。
- 元素类型，用 document.createElement 来创建元素节点，元素节点还有属性要处理，并且要递归的渲染子节点。

vdom 的 render 逻辑就是这样的

```js
if (isTextVdom(vdom)) {
  return mount(document.createTextNode(vdom));
} else if (isElementVdom(vdom)) {
  const dom = mount(document.createElement(vdom.type));

  for (const child of vdom.children) {
    render(child, dom);
  }

  for (const prop in vdom.props) {
    setAttribute(dom, prop, vdom.props[prop]);
  }
  return dom;
}
```

文本的判断就是字符串和数字

```js
function isTextVdom(vdom) {
  return typeof vdom === "string" || typeof vdom === "number";
}
```

元素的判断就是对象，并且 type 为标签名的字符串

```js
function isElementVdom(vdom) {
  return typeof vdom == "object" && typeof vdom.type == "string";
}
```

元素创建出来之后如果有父节点要挂载到父节点，组装成 dom 树：

```js
const mount = parent ? (el) => parent.appendChild(el) : (el) => el;
```

完整的 render 函数

```js
const render = (vdom, parent = null) => {
  const mount = parent ? (el) => parent.appendChild(el) : (el) => el;
  if (isTextVdom(vdom)) {
    return mount(document.createTextNode(vdom));
  } else if (isElementVdom(vdom)) {
    const dom = mount(document.createElement(vdom.type));
    for (const child of vdom.children) {
      render(child, dom);
    }
    for (const prop in vdom.props) {
      setAttribute(dom, prop, vdom.props[prop]);
    }
    return dom;
  }
};
```

其中，元素的 dom 还要设置属性，比如上面 vdom 里有 style 和 onClick 的属性要设置。

style 属性是样式，支持对象，要把对象合并之后设置到 style，而 onClick 属性是事件监听器，用 addEventListener 设置，其余的属性都用 setAttribute 来设置。

```js
const setAttribute = (dom, key, value) => {
  if (typeof value == "function" && key.startsWith("on")) {
    const eventType = key.slice(2).toLowerCase();
    dom.addEventListener(eventType, value);
  } else if (key == "style" && typeof value == "object") {
    Object.assign(dom.style, value);
  } else if (typeof value != "object" && typeof value != "function") {
    dom.setAttribute(key, value);
  }
};
```

3、vdom 的渲染流程

**vdom 会递归的进行渲染，根据类型的不同，元素、文本会分别用 createTextNode、createElement 来递归创建 dom 并组装到一起，其中元素还要设置属性，style、事件监听器和其他属性分别用 addEventListener、setAttribute 等 api 进行设置。通过不同的 api 创建 dom 和设置属性，这就是 vdom 的渲染流程。**

### jsx 编译成 vdom

vdom 写起来太麻烦了，没人会直接写 vdom，一般是通过更友好的 dsl（领域特定语言） 来写，然后编译成 vdom，比如 jsx。

```jsx
const data = {
  item1: "bbb",
  item2: "ddd",
};

const jsx = (
  <ul className="list">
    <li
      className="item"
      style={{ background: "blue", color: "pink" }}
      onClick={() => alert(2)}
    >
      aaa
    </li>
    <li className="item">{data.item1}</li>
    <li className="item">{data.item2}</li>
  </ul>
);

render(jsx, document.getElementById("root"));
```

明显比直接写 vdom 紧凑不少，但是需要做一次编译

```js
module.exports = {
  presets: [
    [
      "@babel/preset-react",
      {
        pragma: "createElement",
      },
    ],
  ],
};
```

编译的产物

```js
const data = {
  item1: "bbb",
  item2: "ddd",
};

const jsx = createElement(
  "ul",
  {
    className: "list",
  },
  createElement(
    "li",
    {
      className: "item",
      style: {
        background: "blue",
        color: "pink",
      },
      onClick: () => alert(2),
    },
    "aaa"
  ),
  createElement(
    "li",
    {
      className: "item",
    },
    data.item1
  ),
  createElement(
    "li",
    {
      className: "item",
    },
    data.item2
  )
);

render(jsx, document.getElementById("root"));

// jsx 返回的就是vdom
const createElement = (type, props, ...children) => {
  return {
    type,
    props,
    children,
  };
};
```

这叫做 render function，它执行的返回值就是 vdom。

vdom 的基础上更进了一步，通过 jsx 来写一些动态逻辑，然后编译成 render function，执行之后产生 vdom。
