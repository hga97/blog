### 正确理解 react-router

#### 理解单页面应用

单页面应用：使用一个 html，一次性加载 js、css 等资源，所有页面都在一个容器页面下，页面切换实质是组件的切换。

#### react-router 路由原理

1、react-router-dom 和 react-router 和 history 库三者什么关系

history 可以理解为 react-router 的核心，也是整个路由原理的核心，里面集成了 popState，history.pushState 等底层路由实现的原理方法。

react-router 可以理解为是 react-router-dom 的核心，里面封装了 Router、Route 和 Switch 等核心组件，实现了从路由的改变到组件的更新的核心功能，项目使用只要一次性引入 react-router-dom 就可以了。

react-router-dom 在 react-router 的核心基础上，添加了用于跳转的 Link 组件，和 history 模式下的 BrowserRouter 和 hash 模式下的 HashRouter 组件等。BrowserRouter 和 HashRouter，用了 history 库中 createBrowserHistory 和 createHashHistory 方法。

### 单页面实现核心原理

单页面应用路由实现原理是，切换 url，监听 url 变化，从而渲染不同的页面组件。

主要的方式有 history 和 hash 模式

#### history 模式原理

1、改变路由

**history.pushState**

```js
history.pushState(state, title, path);
```

- state：一个与指定网址相关的状态对象，popstate 事件触发时，该对象会传入回调函数。如果不需要可填 null
- title：新页面的标题，但是所有浏览器目前都忽略这个值，可填 null
- path：新的网址，必须与当前页面处在同一个域。浏览器的地址栏将显示这个地址。

**history.replaceState**

```js
history.replaceState(state, title, path);
```

这个方法会修改当前的 history 对象记录，history.length 的长度不会改变。

2、监听路由

**popState 事件**

```js
window.addEventListener("popstate", function (e) {
  // 监听改变
});
```

同一个文档的 history 对象出现变化时，就会触发 popstate 事件 history.pushState 可以使浏览器地址改变，但是无需刷新页面。

history.pushState()或者 history.replaceState()不会触发 popstate 事件。popstate 事件只会在浏览器某些行为下触发，比如点击后退、前进按钮或者调用 history.back()、history.forward()和 history.go()方法。

#### hash 模式原理

1、改变路由

window.location.hash

通过 window.location.hash 属性获取和设置 hash 值。

2、监听路由

onhashChange

```js
window.addEventListener("hashchange", function (e) {
  // 监听改变
});
```

### 理解 history 库

history 专注于纪录路由 history 状态，以及 path 改变。**在 history 模式下用 popstate 监听路由变化，在 hash 模式下用 hashchange 监听路由的变化。**

#### createBrowserHistory

```js
const PopStateEvent = "popstate";
const HashChangeEvent = "hashchange";

function createBrowserHistory() {
  // 全局history
  const globalHistory = window.history;

  // 处理路由转换。记录了listens信息。
  const transitionManager = createTransitionManager();

  // 改变location对象，通知组件更新
  const setState = () => {};

  // 处理path改变后，处理popstate变化的回调函数
  const handlePopState = () => {};

  // history.push方法，改变路由，通过全局对象history.pushState改变url，通知router触发更新，替换组件
  const push = () => {};

  // 底层应用事件监听，监听popstate事件
  const listen = () => {};

  return {
    push,
    listen,
  };
}
```

popstate 和 hashchange 是监听路由变化底层方法

1、setState

```js
const setState = (nextState) => {
  Object.assign(history, nextState);
  histroy.length = globalHistroy.length;

  /* 通知每一个listens 路由已经发生变化 */
  transitionManager.notifyListeners(history.location, history.action);
};
```

统一每个 transitionManager 管理的 listener 路由状态已经更新。

2、listen

```js
const listen = (listener) => {
  // 添加listen
  const unlisten = transitionManager.appendlistener(listener);
  checkDOMListeners(1);

  return () => {
    checkDOMListeners(-1);
    unlisten();
  };
};
```

checkDOMListeners

```js
const checkDOMListeners = (delta) => {
  listenerCount += delta;
  if (listenerCount === 1) {
    addEventListener(window, PopStateEvent, handlePopState);
    if (needsHashChangeListener)
      addEventListener(window, HashChangeEvent, handleHashChange);
  } else if (listenerCount === 0) {
    removeEventListener(window, PopStateEvent, handlePopState);
    if (needsHashChangeListener)
      removeEventListener(window, HashChangeEvent, handleHashChange);
  }
};
```

listen 本质通过 checkDOMListeners 的参数 1 或-1 来绑定/解绑 popstate 事件，当路由发生改变的时候，调用处理函数 handlePopState。

3、push

```js
const push = (path, state) => {
  const action = "PUSH";
  // 创建location 对象
  const location = createLocation(path, state, createKey(), history.location);
  // 确定是否能进行路由转换，还在确认的时候又开始了另一个转变，可能会造成异常。
  transitionManager.confirmTransitionTo(
    location,
    action,
    getUserConfirmation,
    (ok) => {
      if (!ok) return;
      const href = createHref(location);
      const { key, state } = location;
      if (canUseHistory) {
        /* 改变 url */
        globalHistory.pushState({ key, state }, null, href);
        if (forceRefresh) {
          window.location.href = href;
        } else {
          /* 改变 react-router location对象, 创建更新环境 */
          setState({ action, location });
        }
      } else {
        window.location.href = href;
      }
    }
  );
};
```

push 流程：生成一个最新的 location 对象，然后通过 window.history.pushState 方法改变浏览器当前路由，最后通过 setState 方法通知 React-Router 更新，并传递当前的 location 对象，由于这次 url 变化，是 history.pushState 产生的，并不会触发 popState 方法，所以需要手动 setState，触发组件更新。

4、handlePopState

```js
cosnt handlePopState = (event) => {
    const location = getDOMLocation(event.state);
    const action = 'POP'

    transitionManager.confirmTransitionTo(location, action, getUserConfirmation, (ok)=>{
        if(ok){
            setState({action, location})
        }else {
            revertPop(location)
        }
    })
}
```

判断 action 类型为 pop，然后 setState，重新加载组件。

#### creatHashHistory

1、监听哈希路由变化

```js
const HashChangeEvent = "hashchange";
const checkDOMListeners = (delta) => {
  listenerCount += delata;
  if (listenerCount === 1) {
    addEventListener(window, HashChangeEvent, handleHashChange);
  } else if (listenerCount === 0) {
    removeEventListener(window, HashChangeEvent, handleHashChange);
  }
};
```

hashchange 来监听 hash 路由的变化

2、改变哈希路由

```js
// 对应push方法
const pushHashPath = (path) => {
  window.location.hash = path;
};

// 对应replace方法
const replaceHashPath = (path) => {
  const hashIndex = window.location.href.indexOf("#");

  window.location.replace(
    window.location.href.slice(0, hashIndex >= 0 ? hashIndex : 0) + "#" + path
  );
};
```

在 hash 模式下，history.push 底层是调用了 window.location.href 来改变路由。
history.replace 底层是去调用 window.location.replace 改变路由。
