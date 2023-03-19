### typeof

typeof 一般被用于判断一个变量的类型，可以利用 typeof 来判断 number、string、object、boolean、function、undefined、symbol 这七种类型。

typeof 在判断一个 object 的数据的时候只能告诉我们这个数据是 object，不能具体到哪一种 object

```js
let s = new String("abc");
typeof s === "object"; // true
s instanceof String; // true
```

**要想判断一个数据具体是哪一种 object，需要利用 instanceof 这个操作符来判断**

#### 原理

js 在底层存储变量的时候，会在变量的机器码的低位 1-3 位存储其类型

- 000：对象
- 010：浮点数
- 100：字符串
- 110：布尔值
- 1：整数
- null：所有机器码均为 0
- undefined：用-2^30 整数来表示

typeof 在判断 null 的时候就出现问题了，由于 null 的所有机器码为 0，因此直接被当做了对象来看待。

用 instanceof 来判断，null 直接被判断为不是 object。

#### Object.prototype.toString

可以利用这个方法来对一个变量的类型来进行比较准确的判断

```js
Object.prototype.toString.call(1); // "[object Number]"

Object.prototype.toString.call("hi"); // "[object String]"

Object.prototype.toString.call({ a: "hi" }); // "[object Object]"

Object.prototype.toString.call([1, "a"]); // "[object Array]"

Object.prototype.toString.call(true); // "[object Boolean]"

Object.prototype.toString.call(() => {}); // "[object Function]"

Object.prototype.toString.call(null); // "[object Null]"

Object.prototype.toString.call(undefined); // "[object Undefined]"

Object.prototype.toString.call(Symbol(1)); // "[object Symbol]"
```

?> Object.prototype.toString 和 对象类型调用 toString 方法  
toString 是 Object 的原型方法。Array、function 等类型作为 Object 的实例，都重写了 toString 方法。不同的对象类型调用 toString 方法时，根据原型链的知识，调用的是对应的重写之后。

### instanceof

主要作用就是判断一个实例是否属于某种类型

```js
let person = function () {};
let nicole = new person();
nicole instanceof person; // true
```

instanceof 也可以判断一个实例是否是其父类型或者祖先类型的实例。

```js
let person = function () {};
let programmer = function () {};
programmer.prototype = new person();
let nicole = new programmer();
nicole instanceof person; // true
nicole instanceof programmer; // true
```

#### 原理

只要右边变量的 prototype 在左边变量的原型链上即可

```js
function new_instance_of(leftVaule, rightVaule) {
  let rightProto = rightVaule.prototype; // 取右表达式的 prototype 值
  leftVaule = leftVaule.__proto__; // 取左表达式的__proto__值
  while (true) {
    if (leftVaule === null) {
      return false;
    }
    if (leftVaule === rightProto) {
      return true;
    }
    leftVaule = leftVaule.__proto__;
  }
}
```
