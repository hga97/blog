### new

new 运算符创建一个用户定义的对象类型的实例或具有构造函数的内置对象类型之一。

new 实现了哪些功能。

- 访问构造函数里的属性
- 访问 prototype 中的属性

```js
function Otaku(name, age) {
  this.name = name;
  this.age = age;

  this.habit = "Games";
}

Otaku.prototype.strength = 60;

Otaku.prototype.sayYourName = function () {
  console.log("I am " + this.name);
};

var person = new Otaku("Kevin", "18");

console.log(person.name); // Kevin
console.log(person.habit); // Games
console.log(person.strength); // 60

person.sayYourName(); // I am Kevin
```

### 初步实现

new 的结果是一个新对象，所以在模拟实现的时候，要建立一个新对象 obj。
obj 具有 Otaku 构造函数里的属性，可以使用 Otaku.apply(obj, arguments) 来给 obj 添加新的属性。

实例的\_\_proto\_\_属性会指向构造函数的 prototype，实例可以访问原型上的属性

第一版

```js
function objectFactory() {
  var obj = new Object(),
    Constructor = [].shift.call(arguments);

  obj.__proto__ = Constructor.prototype;

  Constructor.apply(obj, arguments);

  return obj;
}

function Otaku(name, age) {
  this.name = name;
  this.age = age;

  this.habit = "Games";
}

Otaku.prototype.strength = 60;

Otaku.prototype.sayYourName = function () {
  console.log("I am " + this.name);
};

var person = objectFactory(Otaku, "Kevin", "18");

console.log(person.name); // Kevin
console.log(person.habit); // Games
console.log(person.strength); // 60

person.sayYourName(); // I am Kevin
```

- new Object()的方式新建一个对象 obj
- 取第一个参数，就是传入的构造函数
- 将 obj 的原型指向构造函数，obj 就可以访问到构造函数原型的属性
- 使用 apply，改变构造函数 this 的指向到新建的对象，这样 obj 就可以访问到构造函数中的属性。
- 返回 obj

### 返回值效果实现

构造函数返回了一个对象，在实例 person 中只能访问返回的对象中的属性

```js
function Otaku(name, age) {
  this.strength = 60;
  this.age = age;

  return {
    name: name,
    habit: "Games",
  };
}

var person = new Otaku("Kevin", "18");

console.log(person.name); // Kevin
console.log(person.habit); // Games
console.log(person.strength); // undefined
console.log(person.age); // undefined
```

返回基本类型的值，相当于没有返回值进行处理。

```js
function Otaku(name, age) {
  this.strength = 60;
  this.age = age;

  return "handsome boy";
}

var person = new Otaku("Kevin", "18");

console.log(person.name); // undefined
console.log(person.habit); // undefined
console.log(person.strength); // 60
console.log(person.age); // 18
```

判断返回值是不是一个对象，如果是一个对象，直接返回这个对象。

第二版

```js
function objectFactory() {
  var obj = new Object(),
    Constructor = [].shift.call(arguments);

  obj.__proto__ = Constructor.prototype;

  var ret = Constructor.apply(obj, arguments);

  return typeof ret === "object" ? ret : obj;
}
```
