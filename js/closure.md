### 定义

1、MDN 对闭包的定义为：  
闭包是指那些能够访问自由变量的函数。

2、自由变量：在函数中使用，但既不是函数参数也不是函数的局部变量的变量。

3、闭包 = 函数 + 函数能访问的自由变量

例子

```js
var a = 1;

function foo() {
  console.log(a);
}

foo();
```

foo 函数可以访问变量 a，但是 a 既不是 foo 函数的局部变量，也不是 foo 函数的参数，所以 a 就是自由变量。

函数 foo + foo 函数访问的自由变量 a 构成了一个闭包。

?> 理论的闭包  
《JavaScript 权威指南》中就讲到：从技术的角度讲，所有的 JavaScript 函数都是闭包

?> 实践的闭包  
即使创建它的上下文已经销毁，它仍然存在。（比如，内部函数从父函数中返回）  
在代码中引用了自由变量

### 分析

```js
var scope = "global scope";
function checkscope() {
  var scope = "local scope";
  function f() {
    return scope;
  }
  return f;
}

var foo = checkscope();
foo();
```

代码执行上下文栈和执行上下文的变化情况

1、执行全局代码，创建全局上下文，并将全局上下文压入执行上下文栈。
2、全局初始化，变量对象、scope 和 this
3、执行 checkscope 函数，创建 checkscopeContext 上下文，将其压入执行上下文栈。  
4、初始化 checkscope，变量对象、scope 和 this。
5、执行完毕 checkscope，checkscopeContext 从执行上下文栈中弹出
6、执行 f 函数，创建 f 函数执行上下文，f 执行上下文被压入执行上下文栈
7、f 执行上下文初始化，创建变量对象、scope 和 this
8、f 函数执行完毕，f 函数上下文从执行上下文栈中弹出

思考

f 函数执行的时候，checkscope 函数上下文被销毁了，怎么读取到 checkscope 作用域下的 scope 的值呢

f 执行上下文维护了一个作用域链

```js
fContext = {
  Scope: [AO, checkscopeContext.AO, globalContext.VO],
};
```

f 函数依然可以读取到 checkscopeContext.AO 的值，说明当 f 函数引用了 checkscopeContext.AO 中的值的时候，即使 checkscopeContext 被销毁了，但是 JavaScript 依然会让 checkscopeContext.AO 活在内存中，f 函数依然可以通过 f 函数的作用域链找到它，正是因为 JavaScript 做到了这一点，从而实现了闭包这个概念。

### 闭包实例

```js
var data = [];

for (var i = 0; i < 3; i++) {
  data[i] = function () {
    console.log(i);
  };
}

data[0]();
data[1]();
data[2]();
```

执行到 data[0]函数之前，此时全局上下文的 VO 为

```js
globalContext = {
    VO: {
        data: [...],
        i: 3
    }
}
```

当执行 data[0]函数的时候，data[0]函数的作用域链为

```js
data[0]Context = {
    Scope: [AO, globalContext.VO]
}
```

data[0]Context 的 AO 并没有 i 值，所以会从 globalContext.VO 中查找，i 为 3。data[1]和 data[2]类似。

改成闭包

```js
var data = [];

for (var i = 0; i < 3; i++) {
  data[i] = (function (i) {
    return function () {
      console.log(i);
    };
  })(i);
}

data[0]();
data[1]();
data[2]();
```

执行到 data[0]函数之前，此时全局上下文的 VO 为

```js
globalContext = {
    VO: {
        data: [...],
        i: 3
    }
}
```

当执行 data[0] 函数的时候，data[0] 函数的作用域链发生了改变：

```js
data[0]Context = {
    Scope: [AO, 匿名函数Context.AO globalContext.VO]
}
```

匿名函数执行上下文的 AO 为：

```js
匿名函数Context = {
  AO: {
    arguments: {
      0: 0,
      length: 1,
    },
    i: 0,
  },
};
```

data[0]Context 的 AO 并没有 i 值，所以会沿着作用域链从匿名函数 Context.AO 中查找，这时候就会找 i 为 0，找到了就不会往 globalContext.VO 中查找了，即使 globalContext.VO 也有 i 的值(值为 3)，所以打印的结果就是 0。

data[1] 和 data[2] 是一样的道理。

<!-- todo: 块级作用域===== -->
