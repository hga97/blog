### 路由本质

前端路由：**对 url 进行改变和监听，来让某个 dom 节点显示对应的视图。**

### 路由的区别

浏览器的路由分为两种：

- hash 路由，特征是 url 后面会有#号。
- history 路由，url 和普通路径没有差异。

#### hash

location.hash = 'foo'  
https://www.baidu.com 变成 https://www.baidu.com/#foo

通过 window.addEventListener('hashchange')这个事件，就可以监听到 hash 值的变化。

#### history

```js
history.pushState(state, title[, url])
```

**state**  
代表状态对象，可以给每个路由记录创建自己的状态，并且它还会序列化后保存在用户的磁盘上，以便用户重新启动浏览器后可以将其还原。

**title**  
当前没啥用

**url**  
新的网址

history.pushState({}, '', 'foo')  
https://www.baidu.com 变化为 https://www.baidu.com/foo

路径更新后，浏览器页面不会重新加载？

location.href = 'baidu.com/foo' 这种方式跳转，是会让浏览器重新加载页面并且请求服务器的。
history.pushState 可以让 url 改变，但是不重新加载页面，完全由用户决定如何处理这次 url 改变。

刷新后会 404？

本质上是因为刷新以后是带着 baidu.com/foo 这个页面去请求服务端资源的，但是服务端并没有对这个路径进行任何的映射处理，当然会返回 404，处理方式是让服务端对于"不认识"的页面,返回 index.html，这样这个包含了前端路由相关 js 代码的首页，就会加载你的前端路由配置表，并且此时虽然服务端给你的文件是首页文件，但是你的 url 上是 baidu.com/foo，前端路由就会加载 /foo 这个路径相对应的视图，完美的解决了 404 问题。

history 路由的监听也有点坑，浏览器提供了 window.addEventListener('popstate') 事件，但是**它只能监听到浏览器回退和前进所产生的路由变化，**对于主动的 pushState 却监听不到。

### 实现 react-mini-router

#### 实现 history

- history.push
- history.listen

观察者模式封装了一个简单的 listen api，让用户可以监听到 history.push 所产生的路径改变

```js
let listeners = [];

function listen(fn) {
  listeners.push(fn);

  return () => {
    listeners = listeners.filter((listener) => listener !== fn);
  };
}
```

通过回调，感知到路由的变化，并且在 location 中，提供了 state、pathname 和 search 等关键信息

```js
history.listen((location) => {
  console.log("changed", location);
});
```

实现改变路径的核心方法 push

```js
function push(to, state) {
  location = getNextLoaction(to, state); // 分解成pathname、search等信息
  window.history.pushState(state, "", to); // 原生history的方法改变路由
  listeners.forEach((fn) => fn(location)); // 执行用户传入的监听函数
}
```

history.push('foo')的时候，本质上就是调用 window.history.pushState 去改变路径，并且通知 listen 所挂载的回调函数去执行。

用户点击浏览器后退前进按钮的行为，也需要用 popstate 这个事件来监听，并且执行同样的处理

```js
// 用于处理浏览器前进后退操作
window.addEventListener("popstate", () => {
  location = getLocation();
  listeners.forEach((fn) => fn(location));
});
```

#### 实现 Router

Router 的核心原理就是通过 Provider 把 loction 和 history 等路由关键信息传递给子组件，并且在路由发生变化的时候让子组件可以感知到

```jsx
import React, { useEffect, useState } from "react";
import { history } from "./history";

const RouteContext = React.createContext(null);

export const Router = ({ children }) => {
  const [location, setLocation] = useState(history.location);

  useEffect(() => {
    // 订阅history的变化
    // 一旦路由发生改变就会通知使用了 useContext(RouterContext) 的子组件去重新渲染
    const unlisten = history.listen((location) => {
      setLocation(location);
    });
    return unlisten;
  }, []);

  return (
    <RouteContext.Provider value={{ history, location }}>
      {children}
    </RouteContext.Provider>
  );
};
```

组件初始化的时候利用 history.listen 监听路由的变化，一旦路由发生改变，就会调用 setLocation 去更新 location 并且通过 Provider 传递给子组件。

并且这一步也会触发 Provider 的 value 值的变化，通知所有用 useContext 订阅了 history 和 location 的子组件去重新 render。

#### 实现 Route

Route 组件接受 path 和 children 两个 prop， 本质上就决定了在某个路径下需要渲染什么组件，通过 Router 的 Provider 传递下来的 location 信息拿到当前路径，所以这个组件需要做的就是判断当前的路径是否匹配，渲染对应组件。

```jsx
import { useLocation } from "./hooks";

export const Route = ({ path, children }) => {
  const { pathname } = useLocation();
  const matched = path === pathname;

  if (matched) {
    return children;
  }

  return null;
};
```

#### 实现 useLocation 和 useHistory

```jsx
import { useContext } from "react";
import { RouterContext } from "./Router";

export const useHistory = () => {
  return useContext(RouterContext).history;
};

export const useLocation = () => {
  return useContext(RouterContext).location;
};
```

### demo 验证

```jsx
import { Router, Route, useHistory } from "./src";

const Foo = () => "foo";
const Bar = () => "bar";

const Links = () => {
  const history = useHistory();

  const go = (path) => {
    const state = { name: path };
    history.push(path, state);
  };

  return (
    <div className="demo">
      <button onClick={() => go("foo")}>foo</button>
      <button onClick={() => go("bar")}>bar</button>
    </div>
  );
};

const Demo = () => {
  return (
    <Router>
      <Links />
      <Route path="foo">
        <Foo />
      </Route>
      <Route path="bar">
        <Bar />
      </Route>
    </Router>
  );
};

export default Demo;
```
