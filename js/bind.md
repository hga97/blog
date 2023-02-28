### bind

bind()方法会创建一个函数。当这个新函数被调用时，bind()的第一个参数将作为它运行时的 this,
之后的一序列参数将会在传递的实参前传入作为它的参数。

bind 的特点

- 返回一个函数
- 可以传入参数

### 返回函数的模拟实现

```js
var foo = {
  value: 1,
};

function bar() {
  return this.value;
}

// 返回了一个函数
var bindFoo = bar.bind(foo);

console.log(bindFoo()); // 1
```

this 的指向，可以使用 call 或者 apply 实现。

第一版

```js
Function.prototype.bind2 = function (context) {
  var self = this;
  return function () {
    return self.apply(context); // 绑定的函数可能有返回值
  };
};

var foo = {
  value: 1,
};

function bar() {
  return this.value;
}

var bindFoo = bar.bind2(foo);

console.log(bindFoo()); // 1
```

### 传参的模拟实现

```js
var foo = {
  value: 1,
};

function bar(name, age) {
  console.log(this.value);
  console.log(name);
  console.log(age);
}

var bindFoo = bar.bind(foo, "daisy");
bindFoo("18");
// 1
// daisy
// 18
```

第二版

```js
Function.prototype.bind2 = function (context) {
  var self = this;
  // 获取bind2函数从第二个参数到最后一个参数
  var args = Array.prototype.slice.call(arguments, 1);

  return function () {
    // 这个时候的arguments是指bind返回的函数传入的参数
    var bindArgs = Array.prototype.slice.call(arguments);
    return self.apply(context, args.concat(bindArgs)); // 绑定的函数可能有返回值
  };
};

var foo = {
  value: 1,
};

function bar(name, age) {
  console.log(this.value);
  console.log(name);
  console.log(age);
}

var bindFoo = bar.bind2(foo, "daisy");
bindFoo("18");
// 1
// daisy
// 18
```

### 构造函数效果的模拟实现

bind 还有一个特点，就是

一个绑定函数也能使用 new 操作符创建对象：这种行为就像把原函数当成构造器。
提供的 this 值被忽略，同时调用时的参数被提供给模拟函数。

bind 返回的函数作为构造函数的时候，bind 时指定的 this 值会失效，但传入的参数依然生效。

```js
var value = 2;

var foo = {
  value: 1,
};

function bar(name, age) {
  this.habit = "shopping";
  console.log(this.value);
  console.log(name);
  console.log(age);
}

bar.prototype.friend = "kevin";

var bindFoo = bar.bind(foo, "daisy");
var obj = new bindFoo("18"); // this 指向了 obj

console.log(obj.habit);
console.log(obj.friend);

// undefined
// daisy
// 18
// shopping
// kevin
```

第三版

```js
Function.prototype.bind2 = function (context) {
  var self = this;
  var args = Array.prototype.slice.call(arguments, 1);

  var fBound = function () {
    var bindArgs = Array.prototype.slice.call(arguments);

    return self.apply(
      this instanceof fBound ? this : context,
      args.concat(bindArgs)
    );
  };

  // 修改返回函数的prototype为绑定函数的prototype，实例就可以继承版定函数的原型中的值
  fBound.prototype = this.prototype;
  return fBound;
};

var value = 2;

var foo = {
  value: 1,
};

function bar(name, age) {
  this.habit = "shopping";
  console.log(this.value);
  console.log(name);
  console.log(age);
}

bar.prototype.friend = "kevin";

var bindFoo = bar.bind2(foo, "daisy");
var obj = new bindFoo("18"); // this 指向了 obj

console.log(obj.habit);
console.log(obj.friend);
// undefined
// daisy
// 18
// shopping
// kevin
```

### 构造函数效果的优化实现

fBound.prototype = this.prototype, 直接修改 fBound.prototype 的时候，也会直接修改版定函数的 prototype。

```js
Function.prototype.bind2 = function (context) {
  var self = this;
  var args = Array.prototype.slice.call(arguments, 1);

  var fNOP = function () {};

  var fBound = function () {
    var bindArgs = Array.prototype.slice.call(arguments);

    return self.apply(
      this instanceof fNOP ? this : context,
      args.concat(bindArgs)
    );
  };

  fNOP.prototype = this.prototype;
  fBound.prototype = new fNOP();
  return fBound;
};
```
