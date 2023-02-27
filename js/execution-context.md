### 具体执行分析

详细解析执行上下文栈和执行上下文的具体变化过程

```js
var scope = "global scope";
function checkscope() {
  var scope = "local scope";
  function f() {
    return scope;
  }
  return f();
}
checkscope();
```

1、执行全局代码，创建全局执行上下文，全局上下文被压入执行上下文栈

```js
ECStack = [globalContext];
```

2、全局上下文初始化

```js
globalContext = {
  VO: [global],
  Scope: [globalContext.VO],
  this: globalContext.VO,
};
```

初始化的同时，checkscope 函数被创建，保存作用域链到函数的内部属性[[scope]]

```js
checkscope.[[scope]] = [globalContext.VO] // 初始化，函数内部属性。
```

3、执行 checkscope 函数，创建 checkscope 函数执行上下文，checkscope 函数执行上下文被压入执行上下文栈

```js
ECStack = [checkscopeContext, globalContext]; // 执行入栈
```

4、checkscope 函数执行上下文初始化

- 复制函数[[scope]]属性创建作用域链
- 用 arguments 创建活动对象
- 初始化活动对象，即加入形参、函数声明、变量声明
- 将活动对象压入 checkscope 作用域链顶端

```js
checkscopeContext = {
    AO: {
        arguments: {
            length: 0
        },
        scope: undefined,
        f: reference to function f(){}
    },
    Scope: [AO, globalContext.VO],
    this: undefined
}
```

同时 f 函数被创建，保存作用域链到 f 函数的内部属性[[scope]]

```js
f.[[scope]] = [checkscopeContext.AO, globalContext.VO]
```

5、执行 f 函数，创建 f 函数执行上下文，f 函数执行上下文被压入执行上下文栈

```js
ECStack = [fContext, checkscopeContext, globalContext];
```

6、f 函数执行上下文初始化

- 复制函数[[scope]]属性创建作用域链
- 用 arguments 创建活动对象
- 初始化活动对象，即加入形参、函数声明、变量声明
- 将活动对象压入 f 作用域链顶端

```js
fContext = {
  AO: {
    arguments: {
      length: 0,
    },
  },
  Scope: [AO, checkscopeContext.AO, globalContext.VO],
  this: undefined,
};
```

7、f 函数执行，沿着作用域链查找 scope 值，返回 scope 值

8、f 函数执行完毕，f 函数上下文从执行上下文栈中弹出

```js
ECStack = [checkscopeContext, globalContext];
```

9、checkscope 函数执行完毕，checkscope 执行上下文从执行上下文栈中弹出

```js
ECStack = [globalContext];
```
