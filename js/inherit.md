继承方式的优缺点

### 原型链继承

```js
function Parent() {
  this.name = "kevin";
}

Parent.prototype.getName = function () {
  console.log(this.name);
};

function Child() {}

Child.prototype = new Parent();

var child1 = new Child();

console.log(child1.getName());
```

```js
child1.names.push("yayu");

console.log(child1.names); // ["kevin", "daisy", "yayu"]

var child2 = new Child();

console.log(child2.names); // ["kevin", "daisy", "yayu"]
```

缺点

- 引用类型的属性被所有实例共享
- 在创建 child 的实例时，不能向 Parent 传参

### 构造函数（经典继承）

```js
function Parent() {
  this.names = ["kevin", "daisy"];
}

function Child() {
  Parent.call(this);
}

var child1 = new Child();

child1.names.push("yayu");

console.log(child1.names); // ["kevin", "daisy", "yayu"]

var child2 = new Child();

console.log(child2.names); // ["kevin", "daisy"]
```

```js
function Parent(name) {
  this.name = name;
}

function Child(name) {
  Parent.call(this, name);
}

var child1 = new Child("kevin");

console.log(child1.name); // kevin

var child2 = new Child("daisy");

console.log(child2.name); // daisy
```

优点

- 避免了引用类型的属性被所有实例共享
- 可以在 child 中向 parent 传参

缺点

- 方法都在构造函数中定义，每次创建实例都会创建一遍方法。

### 组合继承

```js
function Parent(name) {
  this.name = name;
  this.colors = ["red", "blue", "green"];
}

Parent.prototype.getName = function () {
  console.log(this.name);
};

function Child(name, age) {
  Parent.call(this, name);
  this.age = age;
}

Child.prototype = new Parent();
Child.prototype.constructor = Child;

var child1 = new Child("kevin", "18");

child1.colors.push("black");

console.log(child1.name); // kevin
console.log(child1.age); // 18
console.log(child1.colors); // ["red", "blue", "green", "black"]

var child2 = new Child("daisy", "20");

console.log(child2.name); // daisy
console.log(child2.age); // 20
console.log(child2.colors); // ["red", "blue", "green"]
```

优点

- 融合原型链继承和构造函数的优点，是 js 中最常用的继承模式。

缺点

- 会调用两次父构造函数

一次设置子类型实例的原型

```js
Child.prototype = new Parent();
```

一次创建子类型实例

```js
var child1 = new Child("kevin", "18");

Parent.call(this, name);

var child1 = new Child("kevin", "18");
// Child.prototype 和 child1 都有一个属性为colors，属性值为['red', 'blue', 'green']。
```

### 原型式继承

```js
function createObj(o) {
  function F() {}
  F.prototype = o;
  return new F();
}
```

ES5 Object.create 的模拟实现，将传入的对象作为创建的对象的原型。

```js
var person = {
  name: "kevin",
  friends: ["daisy", "kelly"],
};

var person1 = createObj(person);
var person2 = createObj(person);

person1.name = "person1"; // 给person1添加了name值，并非修改了原型上的name值，原型链上的值还是kevin。
console.log(person2.name); // kevin

person1.friends.push("taylor");
console.log(person2.friends); // ["daisy", "kelly", "taylor"]
```

缺点

- 包含引用类型的属性值始终都会共享相应的值，这点跟原型链继承一样。

### 寄生式继承

```js
function createObj(o) {
  var clone = Object.create(o);
  clone.sayName = function () {
    console.log("hi");
  };
  return clone;
}
```

缺点

- 跟借用构造函数模式一样，每次创建对象都会创建一遍方法。

### 寄生组合式继承

组合继承调用两次父构造函数

不使用 Child.prototype = new Parent() ，而是间接的让 Child.prototype 访问到 Parent.prototype

```js
function object(o) {
  function F() {}
  F.prototype = o;
  return new F();
}

function prototype(child, parent) {
  var prototype = object(parent.prototype);
  prototype.constructor = child;
  child.prototype = prototype;
}

// 当我们使用的时候：
prototype(Child, Parent);
```

这种方式的高效率体现它只调用了一次 Parent 构造函数，并且因此避免了在 Parent.prototype 上面创建不必要的、多余的属性。与此同时，原型链还能保持不变；因此，还能够正常使用 instanceof 和 isPrototypeOf。开发人员普遍认为寄生组合式继承是引用类型最理想的继承范式。
