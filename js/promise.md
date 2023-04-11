### 基础实现

实现原理，promise 也使用回调函数，只不过把回调封装在了内部，使用上一直通过 then 方法的链式调用，
使得多层的回调嵌套看起来变成了同一层。

#### 基础版

```js
class MyPromise {
  callbacks = [];

  constructor(fn) {
    fn(this._resolve.bind(this));
  }

  then(onFulfilled) {
    this.callbacks.push(onFulfilled);
  }

  _resolve(value) {
    this.callbacks.forEach((fn) => fn(value));
  }
}

let p = new MyPromise((resolve) => {
  setTimeout(() => {
    console.log("done");
    resolve("5秒");
  }, 5000);
}).then((tip) => {
  console.log(tip);
});
```

1、实现原理

- 调用 then 方法，将想要在 promise 异步操作成功时执行的 onFulfilled 放入 callbacks 队列，其实也就是注册回调函数。
- resolve，它接收一个参数 value，代表异步操作返回的结果，当异步操作执行成功后，会调用 resolve 方法，这时候其实真正执行的操作是将 callbacks 队列中的回调一一执行；

2、then 方法可以调用多次

```js
let p = new MyPromise((resolve) => {
  setTimeout(() => {
    console.log("done");
    resolve("5秒");
  }, 5000);
});

p.then((tip) => {
  console.log("then1", tip);
});

p.then((tip) => {
  console.log("then2", tip);
});
```

3、then 能够链式调用

只需要在 then 中 return this 即可.

```js
class MyPromise {
  callbacks = [];

  constructor(fn) {
    fn(this._resolve.bind(this));
  }

  then(onFulfilled) {
    this.callbacks.push(onFulfilled);
    return this;
  }

  _resolve(value) {
    this.callbacks.forEach((fn) => fn(value));
  }
}

let p = new MyPromise((resolve) => {
  setTimeout(() => {
    console.log("done");
    resolve("5秒");
  }, 5000);
})
  .then((tip) => {
    console.log("then1", tip);
  })
  .then((tip) => {
    console.log("then2", tip);
  });
```

#### 加入延迟机制

1、直接 resolve，onFufilled 没执行。

```js
let p = new MyPromise((resolve) => {
  console.log("同步执行");
  resolve("同步执行");
})
  .then((tip) => {
    console.log("then1", tip);
  })
  .then((tip) => {
    console.log("then2", tip);
  });
```

resolve 执行时，onFufilled 还没有注册到，callbacks 是空数组。

2、Promises/A+规范明确要求回调需要通过异步方式执行，用以保证一致可靠的执行顺序。

```js
class MyPromise {
  callbacks = [];
  constructor(fn) {
    fn(this._resolve.bind(this));
  }
  then(onFulfilled) {
    this.callbacks.push(onFulfilled);
    return this;
  }
  _resolve(value) {
    setTimeout(() => {
      //看这里
      this.callbacks.forEach((fn) => fn(value));
    });
  }
}
```

resolve 中增加定时器，通过 setTimeout 机制，将 resolve 中执行回调的逻辑放置到 JS 任务队列末尾，以保证在 resolve 执行时，then 方法的 onFulfilled 已经注册完成。

3、存在问题，在 resolve 执行后，再通过 then 注册上来的 onFulfilled 都没有机会执行了。

```js
let p = new MyPromise((resolve) => {
  console.log("同步执行");
  resolve("同步执行");
})
  .then((tip) => {
    console.log("then1", tip);
  })
  .then((tip) => {
    console.log("then2", tip);
  });

setTimeout(() => {
  p.then((tip) => {
    console.log("then3", tip);
  });
});
```

当执行 then3 时，resolve 已经执行了，then3 的 onFulfilled 没注册上。 所以 then3 的 onFulfilled 没有执行。需要增加状态，保存 resolve 的值。

#### 增加状态

加入状态机制，pending、fulfilled、rejected。

Promise/A+ 规范中明确规定了，pending 可以转化为 fulfilled 或 rejected 并且只能转化一次，
也就是说**如果 pending 转化到 fulfilled 状态，那么就不能再转化到 rejected。并且 fulfilled 和 rejected 状态只能由 pending 转化而来，两者之间不能互相转换。**

```js
class MyPromise {
  callbacks = [];
  state = "pending"; //增加状态
  value = null; //保存结果

  constructor(fn) {
    fn(this._resolve.bind(this));
  }

  then(onFulfilled) {
    if (this.state === "pending") {
      this.callbacks.push(onFulfilled);
    } else {
      onFulfilled(this.value);
    }
    return this;
  }

  _resolve(value) {
    this.state = "fulfilled";
    this.value = value;
    this.callbacks.forEach((fn) => fn(value));
  }
}
```

当增加完状态之后，原先的\_resolve 中的定时器可以去掉了。当 reolve 同步执行时，
虽然 callbacks 为空，回调函数还没有注册上来，但没有关系，因为后面注册上来时，判断状态为 fulfilled，会立即执行回调。

### Promise 链式调用

只是在 then 方法中 return 了 this，使得 Promise 实例可以多次调用 then 方法，
但因为是同一个实例，调用再多次 then 也只能返回相同的一个结果。

真正的链式调用

```js
//使用Promise
function getUserId(url) {
  return new Promise(function (resolve) {
    //异步请求
    http.get(url, function (id) {
      resolve(id);
    });
  });
}

getUserId("some_url")
  .then(function (id) {
    //do something
    return getNameById(id);
  })
  .then(function (name) {
    //do something
    return getCourseByName(name);
  })
  .then(function (course) {
    //do something
    return getCourseDetailByCourse(course);
  })
  .then(function (courseDetail) {
    //do something
  });
```

真正的链式 Promise 是指在当前 Promise 达到 fulfilled 状态后，即开始进行下一个 Promise。

#### 链式调用的实现

```js
//完整的实现
class MyPromise {
  callbacks = [];
  state = "pending"; //增加状态
  value = null; //保存结果

  constructor(fn) {
    fn(this._resolve.bind(this));
  }

  then(onFulfilled) {
    return new Promise((resolve) => {
      this._handle({
        onFulfilled: onFulfilled || null,
        resolve: resolve,
      });
    });
  }

  _handle(callback) {
    if (this.state === "pending") {
      this.callbacks.push(callback);
      return;
    }

    //如果then中没有传递任何东西
    if (!callback.onFulfilled) {
      callback.resolve(this.value);
      return;
    }

    var ret = callback.onFulfilled(this.value);
    callback.resolve(ret);
  }

  _resolve(value) {
    this.state = "fulfilled"; //改变状态
    this.value = value; //保存结果
    this.callbacks.forEach((callback) => this._handle(callback));
  }
}
```

- then 方法，创建并返回了新的 promise 实例，这是串行 promise 的基础，链式调用的根本。
- then 方法传入的形参 onFulfilled 以及创建新 promise 实例时传入的 resolve 放在一起，被 push 到当前 promise 的 callbacks 队列中，
  这是衔接当前 promise 和后邻 promise 的关键所在。

实例讲解

```js
let promiseCount = 1;

//完整的实现 测试Demo
class Promise {
  callbacks = [];
  name = "";
  state = "pending"; //增加状态
  value = null; //保存结果

  constructor(fn) {
    this.name = `Promisse-${promiseCount++}`;
    console.log("[%s]:constructor", this.name);
    fn(this._resolve.bind(this));
  }

  then(onFulfilled) {
    console.log("[%s]:then", this.name);
    return new Promise((resolve) => {
      this._handle({
        onFulfilled: onFulfilled || null,
        resolve: resolve,
      });
    });
  }

  _handle(callback) {
    console.log("[%s]:_handle", this.name, "state=", this.state);

    if (this.state === "pending") {
      this.callbacks.push(callback);
      console.log("[%s]:_handle", this.name, "callbacks=", this.callbacks);
      return;
    }

    //如果then中没有传递任何东西
    if (!callback.onFulfilled) {
      callback.resolve(this.value);
      return;
    }

    var ret = callback.onFulfilled(this.value);
    callback.resolve(ret);
  }

  _resolve(value) {
    console.log("[%s]:_resolve", this.name);
    console.log("[%s]:_resolve", this.name, "value=", value);
    this.state = "fulfilled"; //改变状态
    this.value = value; //保存结果
    this.callbacks.forEach((callback) => this._handle(callback));
  }
}

/**
 * 模拟异步请求
 * @param {*} url
 * @param {*} s
 * @param {*} callback
 */
const mockAjax = (url, s, callback) => {
  setTimeout(() => {
    callback(url + "异步请求耗时" + s + "秒");
  }, 1000 * s);
};

new Promise((resolve) => {
  mockAjax("getUserId", 1, function (result) {
    resolve(result);
  });
}).then((result) => {
  console.log(result);
});

[Promise-1]:constructor
[Promise-1]:then
[Promise-2]:constructor
[Promise-1]:_handle state= pending
[Promise-1]:_handle callbacks= [{…}]
[Promise-1]:_resolve
[Promise-1]:_resolve value= getUserId异步请求耗时1秒
[Promise-1]:_handle state= fulfilled
getUserId异步请求耗时1秒
[Promise-2]:_resolve
[Promise-2]:_resolve value= undefined
```

- new Promise-1 实例，立即执行 mockAjax。
- 调用 Promise-1 的 then 方法，注册 Promise-1 的 onFufilled 函数。
- new Promise-2 实例，立即执行 Promise-1 的\_handle 方法。
- 此时 Promise-1 还是 pending 状态。
- Promise-1 的\_handle 中就把注册的 Promise-1 的 onFufilled 和 Promise-2 的 resolve 保存在 Promise-1 内部的 callbacks。
- 至此当前线程执行结束。返回的是 Promise-2 实例。
- 1s 后，异步请求返回，要改变 Promise-1 的状态和结果，执行 resolve("getUserId 异步请求耗时 1 秒")。
- Promise-1 的值被改变，内容为异步请求返回的结果："getUserId 异步请求耗时 1 秒"。
- Promise-1 的状态变成 fulfilled。
- Promise-1 的 onFulfilled 被执行，打印出了"getUserId 异步请求耗时 1 秒"。
- 调用 Promise-2 的 resolve。
- 改变 Promise-2 的值和状态，因为 Promise-1 的 onFulfilled 没有返回值，所以 Promise-2 的值为 undefined。

#### 链式调用真正的意义

1、return value

执行当前 Promise 的 onFulfilled 时，返回值通过调用第二个 Promise 的 resolve 方法，传递给第二个 Promise，作为第二个 Promise 的值。于是我们考虑如下 Demo:

```js
new Promise(resolve => {
    mockAjax('getUserId', 1, function (result) {
        resolve(result);
    })
}).then(result => {
    console.log(result);
    //对result进行第一层加工
    let exResult = '前缀:' + result;
    return exResult;
}).then(exResult => {
    console.log(exResult);
});

[Promise-1]:constructor
[Promise-1]:then
[Promise-2]:constructor
[Promise-1]:_handle state= pending
[Promise-1]:_handle callbacks= [{…}]
[Promise-2]:then
[Promise-3]:constructor
[Promise-2]:_handle state= pending
[Promise-2]:_handle callbacks= [{…}]
[Promise-1]:_resolve
[Promise-1]:_resolve value= getUserId异步请求耗时1秒
[Promise-1]:_handle state= fulfilled
getUserId异步请求耗时1秒
[Promise-2]:_resolve
[Promise-2]:_resolve value= 前缀:getUserId异步请求耗时1秒
[Promise-2]:_handle state= fulfilled
前缀:getUserId异步请求耗时1秒
[Promise-3]:_resolve
[Promise-3]:_resolve value= undefined
```

- new Promise-1 实例，立即执行 mockAjax。
- 调用 Promise-1 的 then 方法，注册 Promise-1 的 onFufilled 函数。
- new Promise-2 实例，立即执行 Promise-1 的\_handle 方法。
- 此时 Promise-1 还是 pending 状态。
- Promise-1 的\_handle 中就把注册的 Promise-1 的 onFufilled 和 Promise-2 的 resolve 保存在 Promise-1 内部的 callbacks。
- 返回 Promise-2 实例。
- 调用 Promise-2 的 then 方法，注册 Promise-2 的 onFufilled 函数。
- new Promise-3 实例，立即执行 Promise-2 的\_handle 方法。
- 此时 Promise-2 还是 pending 状态。
- Promise-2 的\_handle 中就把注册的 Promise-2 的 onFufilled 和 Promise-3 的 resolve 保存在 Promise-2 内部的 callbacks。
- 至此当前线程执行结束。返回的是 Promise-3 实例。
- 1s 后，异步请求返回，要改变 Promise-1 的状态和结果，执行 resolve("getUserId 异步请求耗时 1 秒")。
- Promise-1 的值被改变，内容为异步请求返回的结果："getUserId 异步请求耗时 1 秒"。
- Promise-1 的状态变成 fulfilled。
- Promise-1 的 onFulfilled 被执行，打印出了"getUserId 异步请求耗时 1 秒"。
- 调用 Promise-2 的 resolve("前缀:getUserId 异步请求耗时 1 秒")。
- Promise-2 的值被改变，内容为异步请求返回的结果："前缀:getUserId 异步请求耗时 1 秒"。
- Promise-2 的状态变成 fulfilled。
- Promise-2 的 onFulfilled 被执行，打印出了"前缀:getUserId 异步请求耗时 1 秒"。
- 调用 Promise-3 的 resolve()。
- 改变 Promise-3 的值和状态，因为 Promise-2 的 onFulfilled 没有返回值，所以 Promise-3 的值为 undefined。

2、return Promise

如果是 Promise 实例，那么就把当前 Promise 实例的状态改变接口重新注册到 resolve 的值对应的 Promise 的 onFulfilled 中，
也就是说当前 Promise 实例的状态要依赖 resolve 的值的 Promise 实例的状态。

```js
 _resolve(value) {

    if (value && (typeof value === 'object' || typeof value === 'function')) {
        var then = value.then;
        if (typeof then === 'function') {
            then.call(value, this._resolve.bind(this));
            return;
        }
    }

    this.state = 'fulfilled';//改变状态
    this.value = value;//保存结果
    this.callbacks.forEach(callback => this._handle(callback));
}


//Demo4
const pUserId = new Promise(resolve => {
  mockAjax('getUserId', 1, function (result) {
    resolve(result);
  })
})
const pUserName = new Promise(resolve => {
  mockAjax('getUserName', 2, function (result) {
    resolve(result);
  })
})

pUserId.then(id => {
  console.log(id)
  return pUserName
}).then(name => {
  console.log(name)
})

[Promise-1]:constructor
[Promise-2]:constructor
[Promise-1]:then
[Promise-3]:constructor
[Promise-1]:_handle state= pending
[Promise-1]:_handle callbacks= [{…}]
[Promise-3]:then
[Promise-4]:constructor
[Promise-3]:_handle state= pending
[Promise-3]:_handle callbacks= [{…}]
[Promise-1]:_resolve
[Promise-1]:_resolve value= getUserId异步请求耗时1秒
[Promise-1]:_handle state= fulfilled
getUserId异步请求耗时1秒
[Promise-3]:_resolve
[Promise-3]:_resolve value= Promise {callbacks: Array(0), name: 'Promise-2', state: 'pending', value: null}
[Promise-2]:then
[Promise-5]:constructor
[Promise-2]:_handle state= pending
[Promise-2]:_handle callbacks= [{…}]
[Promise-2]:_resolve
[Promise-2]:_resolve value= getUserName异步请求耗时2秒
[Promise-2]:_handle state= fulfilled
[Promise-3]:_resolve
[Promise-3]:_resolve value= getUserName异步请求耗时2秒
[Promise-3]:_handle state= fulfilled
getUserName异步请求耗时2秒
[Promise-4]:_resolve
[Promise-4]:_resolve value= undefined
[Promise-5]:_resolve
[Promise-5]:_resolve value= undefined
```

- 赋值 pUserId，new Promise-1 实例，立即执行 mockAjax 函数。
- 赋值 pUserName，new Promise-2 实例，立即执行 mockAjax 函数。
- pUserId.then()，注册 Promise-1 的 onFufilled 函数。
- new Promise-3 实例，立即执行 Promise-1 的\_handle 函数。
- 此时 Promise-1 还是 pending 状态。
- Promise-1 的\_handle 中就把注册的 Promise-1 的 onFufilled 和 Promise-3 的 resolve 保存在 Promise-1 内部的 callbacks。
- 返回 Promise-3 实例。
- pUserId.then().then() 执行 promise-3 的 then 方法, 注册 Promise-3 的 onFufilled 函数。
- new Promise-4 实例，立即执行 Promise-3 的 \_handle 函数。
- 此时 Promise-3 还是 pending 状态。
- Promise-3 的\_handle 中就把注册的 Promise-3 的 onFufilled 和 Promise-4 的 resolve 保存在 Promise-3 内部的 callbacks。
- 至此当前线程执行结束。返回 Promise-4 实例。
- 1s 后，异步请求返回，要改变 Promise-1 的状态和结果，执行 resolve("getUserId 异步请求耗时 1 秒")。
- Promise-1 的值被改变，内容为异步请求返回的结果："getUserId 异步请求耗时 1 秒"。
- Promise-1 的状态变成 fulfilled。
- Promise-1 的 onFulfilled 被执行，打印出了"getUserId 异步请求耗时 1 秒"。
- 调用 Promise-3 的 resolve。
- Promise-3 的值被改变，内容为 pUserName。
- 执行 pUserName.then(), 注册 Promise-2 的 onFufilled 函数（是 Promise-3 的 resolve 函数）。
- new Promise-5 实例，立即执行 Promise2 的 handle 函数。
- 此时 Promise-2 还是 pending 状态。
- Promise-2 的\_handle 中就把注册的 Promise-2 的 onFufilled 和 Promise-5 的 resolve 保存在 Promise-2 内部的 callbacks。
<!-- - 返回 Promise-5 实例。 -->
- 2s 后，异步请求返回，要改变 Promise-2 的状态和结果，执行 resolve("getUserName 异步请求耗时 2 秒")。
- Promise-2 的值被改变，内容为异步请求返回的结果："getUserName 异步请求耗时 2 秒"。
- Promise-2 的状态变成 fulfilled。
- Promise-2 的 onFulfilled 被执行，也就是 Promise-3 的 resolve("getUserName 异步请求耗时 2 秒") 函数被执行。
- Promise-3 的值被改变，内容为异步请求返回的结果："getUserName 异步请求耗时 2 秒"。
- Promise-3 的状态变成 fulfilled。
- Promise-3 的 onFulfilled 被执行，打印出了"getUserName 异步请求耗时 2 秒"。
- 调用 Promise-4 的 resolve。
- 改变 Promise-4 的值和状态，因为 Promise-3 的 onFulfilled 没有返回值，所以 Promise-4 的值为 undefined。
- 调用 Promise-5.resolve。
- 改变 Promise-5 的值和状态，因为 Promise-2 的 onFulfilled 没有返回值，所以 Promise-5 的值为 undefined。

**Promise-3 的 resolve 函数，需要注册到 Promise-2 的 onFulfilled 上。**
这样当 Promise-2 resolve 时，执行 onFullied 函数，间接调用 Priomise-3 的 resolve 函数，从而让 Promise3 的 onFuilled 函数得以执行。
