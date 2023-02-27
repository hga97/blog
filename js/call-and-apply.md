### call

call() 方法在使用一个指定的 this 值和若干个指定的参数值的前提下调用某个函数或方法。

例子

```js
var foo = {
  value: 1,
};

function bar() {
  console.log(this.value);
}

bar.call(foo);
```

1、call 改变了 this 的指向，指向到 foo  
2、bar 函数执行了

### 模拟实现第一步

模拟步骤

- 将函数设为对象的属性
- 执行该函数
- 删除改函数

第一版

```js
Function.prototype.call2 = function (context) {
  context.fn = this; // 获取调用call的函数
  context.fn(); // 执行，this指向了context
  delete context.fn; // 删掉
};

var foo = {
  value: 1,
};

function bar() {
  console.log(this.value);
}

bar.call2(foo); // 1
```

### 模拟实现第二步

call 函数还能给定参数执行函数。

```js
var foo = {
  value: 1,
};

function bar(name, age) {
  console.log(name);
  console.log(age);
  console.log(this.value);
}

bar.call(foo, "kevin", 18);
// kevin
// 18
// 1
```

从 Argument 对象中取第二个到最后一个参数。

```js
var agrs = [];
for (var i = 1, len = arguments.length; i < len; i++) {
  args.push("arguments[" + i + "]");
}

// args为 ["arguments[1]", "arguments[2]" ...]
```

将参数数组放到要执行的函数的参数里面去

```js
eval("context.fn(" + args + ")");
```

第二版

```js
Function.prototype.call2 = function (context) {
  context.fn = this;
  var args = [];
  // 获取参数
  for (var i = 1, len = arguments.length; i < len; i++) {
    args.push("arguments[" + i + "]");
  }
  // 执行传参
  eval("context.fn(" + args + ")");
  delete context.fn;
};

var foo = {
  value: 1,
};

function bar(name, age) {
  console.log(name);
  console.log(age);
  console.log(this.value);
}

bar.call2(foo, "kevin", 18);
// kevin
// 18
// 1
```

### 模拟实现第三步

1、this 参数可以传 null，当为 null，视为 window

```js
var value = 1;

function bar() {
  console.log(this.value);
}

bar.call(null); // 1
```

2、函数可以有返回值的

```js
var obj = {
  value: 1,
};

function bar(name, age) {
  return {
    value: this.value,
    name: name,
    age: age,
  };
}

console.log(bar.call(obj, "kevin", 18));

// {value: 1, name: 'kevin', age: 18}
```

第三版

```js
Function.prototype.call2 = function (context) {
  context = context || window; // 为null时，将函数挂在window上
  context.fn = this;

  var args = [];
  for (var i = 1, len = arguments.length; i < len; i++) {
    args.push("arguments[" + i + "]");
  }

  var result = eval("context.fn(" + args + ")"); // 获取函数的返回值

  delete context.fn;
  return result;
};

// 测试一下
var value = 2;

var obj = {
  value: 1,
};

function bar(name, age) {
  console.log(this.value);
  return {
    value: this.value,
    name: name,
    age: age,
  };
}

bar.call2(null); // 2

console.log(bar.call2(obj, "kevin", 18));
// {value: 1, name: 'kevin', age: 18}
```

### apply 的模拟实现

```js
Function.prototype.apply2 = function (context, arr) {
  var context = Object(context) || window;
  context.fn = this;

  var result;
  if (!arr) {
    result = context.fn();
  } else {
    var args = [];
    for (var i = 0, len = arr.length; i < len; i++) {
      args.push("arr[" + i + "]");
    }
    result = eval("context.fn(" + args + ")");
  }

  delete context.fn;
  return result;
};
```
