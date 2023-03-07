function 组件不能做继承，因为 function 本来就没这个特性，所以是提供了一些 api 供函数使用，这些 api 会在内部的一个数据结构上挂载一些函数和值，并执行相应的逻辑，通过这种方式实现了 state 和类似 class 组件的生命周期函数的功能，这种 api 就叫做 hooks。

### hooks 存储位置

hooks 就是通过把数据挂载到组件对应的 fiber 节点上来实现的。

fiber 节点的 memorizedState 就是保存 hooks 数据的地方。

它是一个通过 next 串联的链表。

```jsx
import { useState, useCallback, useEffect, useRef, useMemo } from "react";

function App() {
  const [name, setName] = useState("guang");
  useState("dong");

  const handler = useCallback(
    (evt) => {
      setName("dong");
    },
    [1]
  );

  useEffect(() => {
    console.log(1);
  });

  useRef(1);

  useMemo(() => {
    return "guang and dong";
  });

  return (
    <div className="App">
      <header className="App-header">
        <p onClick={handler}>{name}</p>
      </header>
    </div>
  );
}

export default App;
```

这就是 hooks 存取数据的地方，执行的时候各自在自己的那个 memorizedState 上存取数据，完成各种逻辑，这就是 hooks 的原理。

### memorizedState 链表创建

链表创建的过程，也就是 mountXxx。链表只需要创建一次，后面只需要 update。

所以第一次调用 useState 会执行 mountState，后面再调用 useState 会执行 updateState。

实现: 创建对应的 memorizedState 对象，然后用 next 串联起来。

```js
function mountWorkInProgressHook() {
  var hook = {
    memoizedState: null,
    baseState: null,
    baseQueue: null,
    queue: null,
    next: null,
  };

  if (workInProgressHook === null) {
    currentlyRenderFiber$1.memoizedState = workInProgressHook = hook;
  } else {
    workInProgressHook = workInProgressHook.next = hook;
  }

  return workInProgressHook;
}
```

所有 hooks api 都是基于 fiber 节点上的 memorizedState 链表来存取数据并完成各自的逻辑的。

当然，创建这样的数据结构还是为了使用的，每种 hooks api 都有不同的使用这些 memorizedState 数据的逻辑，有的比较简单，比如 useRef、useCallback、useMemo，有的没那么简单，比如 useState、useEffect。

### useRef

```js
function mountRef(initialValue) {
  var hook = mountWorkInProgressHook();
  var ref = {
    current: initialValue,
  };

  {
    Object.seal(ref);
  }

  hook.memoizedState = ref;
  return ref;
}

function updateRef(initialValue) {
  var hook = updateWorkInProgressHook();
  return hook.memoizedState;
}
```

可以看到是把传进来的 value 包装了一个有 current 属性的对象，冻结了一下，然后放在 memorizedState 属性上。

后面 update 的时候，没有做任何处理，直接返回这个对象。

useRef 的功能：**useRef 可以保存一个数据的引用，这个引用不可变**

### useCallback

```js
function mountCallback(callback, deps) {
  var hook = mountWorkInProgressHook();
  var nextDeps = deps === undefined ? null : deps;
  hook.memoizedState = [callback, nextDeps];
  return callback;
}

function updateCallback(callback, deps) {
  var hook = updateWorkInProgressHook();
  var nextDeps = deps === undefined ? null : deps;
  var prevState = hook.memoizedState;

  if (prevState !== null) {
    if (nextDeps !== null) {
      var prevDeps = prevState[1];

      if (areHookInputsEqual(nextDeps, prevDeps)) {
        return prevState[0];
      }
    }
  }

  hook.memoizedState = [callback, nextDeps];
  return callback;
}
```

更新的时候把之前的那个 memorizedState 取出来，和新传入的 deps 做下对比，如果没变，那就返回之前的回调函数，也就是 prevState[0]。

如果变了，那就创建一个新的数组，第一个元素是传入的回调函数，第二个是传入的 deps。

useCallback 的功能：**可以实现函数的缓存，如果 deps 没变就不会创建新的，否则才会返回新传入的函数**

### useMemo

```js
function mountMemo(nextCreate, deps) {
  var hook = mountWorkInProgressHook();
  var nextDeps = deps === undefined ? null : deps;
  var nextValue = nextCreate();
  hook.memoizedState = [nextValue, nextDeps];
  return nextValue;
}

function updateMemo(nextCreate, deps) {
  var hook = updateWorkInProgressHook();
  var nextDeps = deps === undefined ? null : deps;
  var prevState = hook.memoizedState;

  if (prevState !== null) {
    if (nextDeps !== null) {
      var prevDeps = prevState[1];

      if (areHookInputsEqual(nextDeps, prevDeps)) {
        return prevState[0];
      }
    }
  }

  var nextValue = nextCreate();
  hook.memoizedState = [nextValue, nextDeps];
  return nextValue;
}
```

更新的时候也是取出之前的 memorizedState，和新传入的 deps 做下对比，如果没变，就返回之前的值，也就是 prevState[0]。

如果变了，创建一个新的数组放在 memorizedState，第一个元素是新传入函数的执行结果，第二个元素是 deps。

useMemo 的功能：可以实现函数执行结果的缓存，如果 deps 没变，就直接拿之前的，否则才会执行函数拿到最新的结果返回。

### useState

### useEffect

### 自定义 hook

```jsx
function useName() {
  const [name, setName] = useState("1");
  useState("1");
  return [name, setName];
}
```

其实就是个函数调用，我们可以把上面的 hooks 放到 xxx 函数里，然后在 function 组件里调用，对应的 hook 链表是一样的。

只不过一般我们会使用 React 提供的 eslint 插件，lint 了这些函数必须以 use 开头，但其实不用也没事，它们和普通的函数封装没有任何区别。
