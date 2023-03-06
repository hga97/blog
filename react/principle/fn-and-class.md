### 实现组件渲染

[babel 转 jsx](https://www.babeljs.cn/repl#?browsers=defaults%2C%20not%20ie%2011%2C%20not%20ie_mob%2011&build=&builtIns=false&corejs=3.6&spec=false&loose=false&code_lz=Q&debug=false&forceAllTransforms=false&shippedProposals=false&circleciRepo=&evaluate=false&fileSize=false&timeTravel=false&sourceType=module&lineWrap=true&presets=env%2Creact%2Cstage-2&prettier=false&targets=&version=7.21.2&externalPlugins=&assumptions=%7B%7D)

```jsx
function Item(props) {
  return (
    <li className="item" style={props.style} onClick={props.onClick}>
      {props.children}
    </li>
  );
}

function List(props) {
  return (
    <ul className="list">
      {props.list.map((item, index) => {
        return (
          <Item
            style={{ background: item.color }}
            onClick={() => alert(item.text)}
          >
            {item.text}
          </Item>
        );
      })}
    </ul>
  );
}

const list = [
  {
    text: "aaa",
    color: "blue",
  },
  {
    text: "ccc",
    color: "orange",
  },
  {
    text: "ddd",
    color: "red",
  },
];

render(<List list={list} />, document.getElementById("root"));
```

转译为

```js
"use strict";

function Item(props) {
  return createElement(
    "li",
    {
      className: "item",
      style: props.style,
      onClick: props.onClick,
    },
    props.children
  );
}

function List(props) {
  return createElement(
    "ul",
    {
      className: "list",
    },
    props.list.map((item, index) => {
      return createElement(
        Item,
        {
          style: {
            background: item.color,
          },
          onClick: () => alert(item.text),
        },
        item.text
      );
    })
  );
}

const list = [
  {
    text: "aaa",
    color: "blue",
  },
  {
    text: "ccc",
    color: "orange",
  },
  {
    text: "ddd",
    color: "red",
  },
];

render(
  createElement(List, {
    list: list,
  }),
  document.getElementById("root")
);
```

我们在 render 函数里处理下函数组件的渲染：

```js
if (isComponentVdom(vdom)) {
  const props = Object.assign({}, vdom.props, {
    children: vdom.children,
  });

  const componentVdom = vdom.type(props);
  return render(componentVdom, parent);
}

function isComponentVdom(vdom) {
  return typeof vdom.type == "function";
}
```

如果 vdom 是一个组件，那么就创建 props 作为参数传入（props 要加上 children），执行该函数组件，拿到返回的 vdom 再渲染。

### class 组件

```jsx
function Item(props) {
  return (
    <li className="item" style={props.style} onClick={props.onClick}>
      {props.children}
    </li>
  );
}

class List extends Component {
  constructor(props) {
    super();
    this.state = {
      list: [
        {
          text: "aaa",
          color: "blue",
        },
        {
          text: "bbb",
          color: "orange",
        },
        {
          text: "ccc",
          color: "red",
        },
      ],
      textColor: props.textColor,
    };
  }

  render() {
    return (
      <ul className="list">
        {this.state.list.map((item, index) => {
          return (
            <Item
              style={{ background: item.color, color: this.state.textColor }}
              onClick={() => alert(item.text)}
            >
              {item.text}
            </Item>
          );
        })}
      </ul>
    );
  }
}

render(<List textColor={"pink"} />, document.getElementById("root"));
```

转译为

```js
"use strict";

function Item(props) {
  return createElement(
    "li",
    {
      className: "item",
      style: props.style,
      onClick: props.onClick,
    },
    props.children
  );
}

class List extends Component {
  constructor(props) {
    super();
    this.state = {
      list: [
        {
          text: "aaa",
          color: "blue",
        },
        {
          text: "bbb",
          color: "orange",
        },
        {
          text: "ccc",
          color: "red",
        },
      ],
      textColor: props.textColor,
    };
  }
  render() {
    return createElement(
      "ul",
      {
        className: "list",
      },
      this.state.list.map((item, index) => {
        return createElement(
          Item,
          {
            style: {
              background: item.color,
              color: this.state.textColor,
            },
            onClick: () => alert(item.text),
          },
          item.text
        );
      })
    );
  }
}

render(
  createElement(List, {
    textColor: "pink",
  }),
  document.getElementById("root")
);
```

渲染 vdom 的时候，如果是类组件，单独处理下：

```js
if (isComponentVdom(vdom)) {
  const props = Object.assign({}, vdom.props, {
    children: vdom.children,
  });

  if (Component.isPrototypeOf(vdom.type)) {
    const instance = new vdom.type(props);

    instance.componentWillMount(); // 生命周期

    const componentVdom = instance.render();
    instance.dom = render(componentVdom, parent);

    instance.componentDidMount(); // 生命周期

    return instance.dom;
  } else {
    const componentVdom = vdom.type(props);
    return render(componentVdom, parent);
  }
}
```

如果 vdom 是 Component，就传入 props 创建实例，然后调用 render 拿到 vdom 再渲染。
