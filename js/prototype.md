### 构造函数创建对象

使用构造函数创建一个对象

```js
function Person() {}

const person = new Person();

person.name = "kevin";
```

**Person 就是一个构造函数，使用 new 创建了一个实例对象 person**

### prototype（构造函数与原型）

函数有一个 prototype 属性

```js
function Person() {}
Person.prototype.name = "kevin";

const person1 = new Person();
const person2 = new Person();
```

**函数的 prototype 属性指向了一个对象，这个对象正是调用该构造函数而创建的实例原型，也就是这个例子中的 person1 和 person2 的原型**

?> 什么是原型  
每一个 js 对象（null 除外）在创建的时候就会与之关联另一个对象，这个对象就是我们所说的原型，每一个对象都会从原型“继承”属性。

### \_\_proto\_\_（实例与原型）

**每一个 js 对象(除了 null)都具有的一个属性，叫\_\_proto\_\_，这个属性会指向该对象的原型。**

```js
function Person() {}
Person.prototype.name = "kevin";

const person = new Person();

console.log(person.__proto__ === Person.prototype); // true
```

### constructor（原型与构造函数）

**每个原型都有一个 constructor 属性指向关联的构造函数**

```js
function Person() {}
Person.prototype.name = "kevin";

console.log(Person === Person.prototype.constructor); // true
```

综上得出

```js
function Person() {}
const person = new Person();

console.log(person.__proto__ == Person.prototype); // true
// ES5的方法,获得对象的原型
console.log(Object.getPrototypeOf(person) === Person.prototype); // true
console.log(Person.prototype.constructor == Person); // true
```

### 实例与原型

**当读取实例的属性时，如果找不到，就会查找与对象关联的原型中的属性，如果还查不到，就去找原型的原型，一值找到最顶层为止。**

```js
function Person() {}
Person.prototype.name = "kevin";

const person = new Person();

person.name = "Daisy";
console.log(person.name); // Daisy

delete person.name;
console.log(person.name); // kevin
```

给实例对象 person 添加了 name 属性，打印 person.name 的时候，结果自然为 Daisy。  
删除了 person 的 name 属性时，读取 person.name，从 person 对象中找不到 name 属性。就会从 person 的原型，就是 person.\_\_proto\_\_ ，也就是 Person.prototype 中查找，找到了 name 属性，结果为 Kevin。

附张图可以清晰看出，name 属性的查找过程：

![在这里插入图片描述](https://img-blog.csdnimg.cn/e770583494494060ad3fecccd715c759.png)

如果没找到呢？原型的原型又是什么？

### 原型的原型

```js
Object.prototype.name = "Tom";

function Person() {}
const person = new Person();

person.name = "Daisy";
console.log(person.name); // Daisy
console.log(person);

delete person.name;
console.log(person.name); // Tom
```

其实原型对象是通过 Object 构造函数生成的，结合之前所讲，实例的\_\_proto\_\_指向构造函数 Object 的 prototype。

![在这里插入图片描述](https://img-blog.csdnimg.cn/544850f5c1fd496b9120cc1f3e6e698f.png)

### 原型链

那 Object.prototype 的原型呢？

```js
console.log(Object.prototype.__proto__ === null); // true
```

null 表示没有对象，即该处不应该有值。

所以 Object.prototype.\_\_proto\_\_ 的值为 null 跟 Object.prototype 没有原型，其实表达了一个意思。

所以查找属性的时候查到 Object.prototype 就可以停止查找了。

```js
function Person() {}
const person = new Person();

person.name = "Daisy";
console.log(person.name); // Daisy
console.log(person);

delete person.name;
console.log(person.name); // undefined
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/c7582fb14fdc41aebb031f6a2f9d06c9.png)
