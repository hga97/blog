### 原始类型：string, number和boolean

string：表示字符串，比如 "hello,world"

number表示数字，比如42（没有int或者float，所有的数字类型都是number）

boolean表示布尔值，两个值true和false

### 数组（Array）

声明一个类似于 [1, 2, 3] 的数组类型，你需要用到语法 number[]。

这个语法可以适用于任何类型（举个例子，string[] 表示一个字符串数组）。

你也可能看到这种写法 Array\<number>，是一样的。我们会在泛型章节为大家介绍 T\<U> 语法。

### any

TypeScript 有一个特殊的类型，any，当你不希望一个值导致类型检查错误的时候，就可以设置为 any 。

当一个值是 any 类型的时候，你可以获取它的任意属性 (也会被转为 any 类型)，或者像函数一样调用它，
把它赋值给一个任意类型的值，或者把任意类型的值赋值给它，再或者是其他语法正确的操作。

```ts

let obj: any = { x: 0 };
// None of the following lines of code will throw compiler errors.
// Using `any` disables all further type checking, and it is assumed 
// you know the environment better than TypeScript.

obj.foo();
obj();
obj.bar = 100;
obj = "hello";
const n: number = obj;

```

noImplicitAny

如果你没有指定一个类型，TypeScript 也不能从上下文推断出它的类型，编译器就会默认设置为 any 类型。

如果你总是想避免这种情况，毕竟 TypeScript 对 any 不做类型检查，你可以开启编译项 noImplicitAny，
当被隐式推断为 any 时，TypeScript 就会报错。

### 变量上的类型注解

选择性的添加一个类型注解，显式指定变量的类型

```ts
let myName: string = "Alice";

```

类型注解往往跟在要被声明类型的内容后面。

不过大部分时候，这不是必须的。因为 TypeScript 会自动推断类型。

```ts
let myName = "Alice";

let myName: string

```

### 函数（Function）

函数是 JavaScript 传递数据的主要方法。TypeScript 允许你指定函数的输入值和输出值的类型。

#### 参数类型注解

当你声明一个函数的时候，你可以在每个参数后面添加一个类型注解，声明函数可以接受什么类型的参数。

```ts
// Parameter type annotation
function greet(name: string) {
  console.log("Hello, " + name.toUpperCase() + "!!");
}

当参数有了类型注解的时候，TypeScript 便会检查函数的实参：
greet(42);
// 类型“number”的参数不能赋给类型“string”的参数。ts(2345)

```

即便你对参数没有做类型注解，TypeScript 依然会检查传入参数的数量是否正确

#### 返回值类型注解

你也可以添加返回值的类型注解。返回值的类型注解跟在参数列表后面：

```ts
function getFavoriteNumber(): number {
  return 26;
}

```

跟变量类型注解一样，你也不需要总是添加返回值类型注解，TypeScript 会基于它的 return 语句推断函数的返回类型。（自动推断类型）

像这个例子中，类型注解写和没写都是一样的，但一些代码库会显式指定返回值的类型，可能是因为需要编写文档，或者阻止意外修改，亦或者仅仅是个人喜好。

#### 匿名函数

匿名函数有一点不同于函数声明，当 TypeScript 知道一个匿名函数将被怎样调用的时候，匿名函数的参数会被自动的指定类型。

```ts
// const names: string[]
const names = ['Alice', 'Bob', 'Eve'];
 
// (parameter) s: string
names.forEach(function (s) {
  console.log(s.toUppercase());
  // Property 'toUppercase' does not exist on type 'string'. Did you mean 'toUpperCase'?
});
 
```

尽管参数 s 并没有添加类型注解，但 TypeScript 根据 forEach 函数的类型，以及传入的数组的类型，最后推断出了 s 的类型。

这个过程被称为上下文推断（contextual typing），因为正是从函数出现的上下文中推断出了它应该有的类型。

### 对象类型

定义一个对象类型，我们只需要简单的列出它的属性和对应的类型。

```ts
function printCoord(pt: {x: number;y: number;}): void

// The parameter's type annotation is an object type
function printCoord(pt: { x: number; y: number }) {
  console.log("The coordinate's x value is " + pt.x);
  console.log("The coordinate's y value is " + pt.y);
}
printCoord({ x: 3, y: 7 });

```

我们给参数添加了一个类型，该类型有两个属性, x 和 y，两个都是 number 类型。

#### 可选属性

对象类型可以指定一些甚至所有的属性为可选的，你只需要在属性名后添加一个 ?:

```ts
function printName(obj: { first: string; last?: string }) {
  // ...
}
// Both OK
printName({ first: "Bob" });
printName({ first: "Alice", last: "Alisson" });

```

在 JavaScript中，如果你获取一个不存在的属性，你会得到一个 undefined 而不是一个运行时错误。因此，当你获取一个可选属性时，你需要在使用它前，先检查一下是否是undefined。

```ts
function printName(obj: { first: string; last?: string }) {

  console.log(obj.last.toUpperCase());
  // (property) last?: string | undefined
  // 对象可能为“未定义”。ts(2532)

  if (obj.last !== undefined) {
    // OK
    console.log(obj.last.toUpperCase());
  }
 
  // A safe alternative using modern JavaScript syntax:
  console.log(obj.last?.toUpperCase());
}
```

### 联合类型

ts类型系统允许你使用一系列的操作符，基于已经存在的类型构建新的类型。
现在我们知道如何编写一些基础的类型了，是时候把它们组合在一起了。

#### 定义一个联合类型

第一种组合类型的方式是使用联合类型，一个联合类型是由两个或者更多类型组成的类型，
表示值可能是这些类型中的任意一个。这其中每个类型都是联合类型的成员

让我们写一个函数，用来处理字符串或者数字：

```ts
function printId(id: number | string) {
  console.log("Your ID is: " + id);
}
// OK
printId(101);
// OK
printId("202");
// Error
printId({ myID: 22342 });
// 类型“{ myID: number; }”的参数不能赋给类型“string | number”的参数。

```

#### 使用联合类型

提供一个符合联合类型的值很容易，你只需要提供符合任意一个联合成员类型的值即可。那么在你有了一个联合类型的值后，你该怎样使用它呢？

使用时必须对每个联合的成员都是有效的。举个例子，如果你有一个联合类型 string | number , 你不能使用只存在 string 上的方法：

```ts
function printId(id: number | string) {
  console.log(id.toUpperCase());
    // Property 'toUpperCase' does not exist on type 'string | number'.
    // Property 'toUpperCase' does not exist on type 'number'.
}

```

解决方案是用代码收窄联合类型，就像你在 JavaScript 没有类型注解那样使用。
当 TypeScript 可以根据代码的结构推断出一个更加具体的类型时，类型收窄就会出现。

举个例子，TypeScript 知道，对一个 string 类型的值使用 typeof 会返回字符串 "string"：

```ts
function printId(id: number | string) {
  if (typeof id === "string") {
    // In this branch, id is of type 'string'
    console.log(id.toUpperCase());
  } else {
    // Here, id is of type 'number'
    console.log(id);
  }
}

function welcomePeople(x: string[] | string) {
  if (Array.isArray(x)) {
    // Here: 'x' is 'string[]'
    console.log("Hello, " + x.join(" and "));
  } else {
    // Here: 'x' is 'string'
    console.log("Welcome lone traveler " + x);
  }
}

```

如果联合类型里的每个成员都有一个属性，举个例子，数组和字符串都有 slice 方法，你就可以直接使用这个属性，而不用做类型收窄：

```ts
// Return type is inferred as number[] | string
function getFirstThree(x: number[] | string) {
  return x.slice(0, 3);
}

```