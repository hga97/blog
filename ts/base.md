### 基础

js动态类型检查

```js
const message = "Hello World!";

message.toLowerCase();
message(); // TypeError: message is not a function

```

**运行时才发现错误**：当我们运行代码的时候，JavaScript会在运行时先算出值的类型（type），然后再决定干什么。
所谓值的类型，也包括了这个值有什么行为和能力。当然 TypeError 也会暗示性的告诉我们一点，比如在这个例子里，
它告诉我们字符串 Hello World 不能作为函数被调用。

**JavaScript 仅仅提供了动态类型（dynamic typing），这需要你先运行代码然后再看会发生什么。**



### 静态类型检查

理想情况下，我们应该有一个工具可以帮助我们，**在代码运行之前就找到错误。这就是静态类型检查器。**比如 TypeScript 做的事情。
静态类型系统（Static types systems）描述了值应有的结构和行为。
一个像 TypeScript 的类型检查器会利用这个信息，并且在可能会出错的时候告诉我们：

```ts
const message = "hello!";
 
message();

// This expression is not callable.
// Type 'String' has no call signatures.

```


### 非异常失败

一个静态类型需要标记出哪些代码是一个错误，哪怕实际生效的js并不会立刻报错。

1、对象不存在该属性

```ts

const user = {
  name: "Daniel",
  age: 26,
};
 
user.location;// 类型“{ name: string; age: number; }”上不存在属性“location”。ts(2339)

```

2、拼写错误

```ts

const announcement = "Hello World!";
 
announcement.toLocaleLowercase();// 属性“toLocaleLowercase”在类型“"Hello World!"”上不存在。你是否指的是“toLocaleLowerCase”?ts(2551)

```


3、函数未被调用

```ts

function flipCoin() {
    return Math.random < 0.5; // 运算符“<”不能应用于类型“() => number”和“number”。ts(2365)
}

```

4、基本的逻辑错误

```ts

const value = Math.random() < 0.5 ? "a" : "b";

if (value !== "a") {
  // ...
} else if (value === "b") { //此条件将始终返回 "false"，因为类型 ""a"" 和 ""b"" 没有重叠。ts(2367)

}

```


### 类型工具

补全、错误信息、快速修复功能和导航功能。


### tsc TS编译器

tsc 会把ts文件编译成一个纯js文件


### 报错时仍产出文件

tsc --noEmitOnError hello.ts
报错，hello.js 并不会得到更新


### 显示类型

```ts

function greet(person: string, date: Date) {
  console.log(`Hello ${person}, today is ${date.toDateString()}!`);
}

// person 是一个 string 类型
// date 是一个 Date 对象
// 给 person 和 date 添加了类型注解（type annotations），描述 greet 函数可以支持传入什么样的值。

function greet(person: string, date: Date) {
  console.log(`Hello ${person}, today is ${date.toDateString()}!`);
}
 
greet("Maddison", Date()); // 类型“string”的参数不能赋给类型“Date”的参数。ts(2345)

greet("Maddison", new Date()); // yes

```


记住，我们并不需要总是写类型注解，大部分时候，TypeScript 可以自动推断出类型：

```ts

let msg = "hello there!";
// let msg: string

// 如果类型系统可以正确的推断出类型，最好就不要手动添加类型注解了。

```


### 类型抹除

例子代码编译成

```ts

"use strict";
function greet(person, date) {
    console.log("Hello " + person + ", today is " + date.toDateString() + "!");
}
greet("Maddison", new Date());

// person 和 date 参数不再有类型注解
// 模板字符串，即用 ``` 包裹的字符串被转换为使用 + 号连接
```

类型注解并不是 JavaScript 的一部分。所以并没有任何浏览器或者运行环境可以直接运行 TypeScript 代码。
这就是为什么 TypeScript 需要一个编译器，它需要将 TypeScript 代码转换为 JavaScript 代码，
然后你才可以运行它。所以大部分 TypeScript 独有的代码会被抹除，在这个例子中，像我们的类型注解就全部被抹除了。


### 降级

```ts

`Hello ${person}, today is ${date.toDateString()}!`;

->

"Hello " + person + ", today is " + date.toDateString() + "!";

```


这是因为模板字符串是 ECMAScript2015（也被叫做 ECMAScript 6 ,ES2015, ES6 等）里的功能，TypeScript 可将新版本的代码编译为老版本的代码，比如 ECMAScript3 或者 ECMAScript5 。这个将高版本的 ECMAScript 语法转为低版本的过程就叫做降级（downleveling） 。


以使用 target (opens new window)选项转换为比较新的一些版本

```ts
tsc --target es2015 hello.ts

function greet(person, date) {
  console.log(`Hello ${person}, today is ${date.toDateString()}!`);
}

greet("Maddison", new Date());

```

尽管默认的目标是 ES3 版本，但是大多数的浏览器都已经支持 ES2015 了，因此大部分开发者可以安全的指定为 ES2015 或者更新的版本，除非你非要兼容某个问题浏览器。


### 严格模式

TypeScript 提供的形式更像是一个刻度盘，你越是转动它，TypeScript 就会检查越多的内容。
这需要一点额外的工作，但是是值得的，它可以带来更全面的检查和更准确的工具功能。如果可能的话，新项目应该始终开启这些严格设置。

TypeScript 有几个严格模式设置的开关。除非特殊说明，文档里的例子都是在严格模式下写的。
CLI 里的 strict 配置项，或者 tsconfig.json 中的 "strict": true 可以同时开启，
也可以分开设置。在这些设置里，你最需要了解的是 noImplicitAny 和 strictNullChecks


### noImplicitAny

在某些时候，TypeScript 并不会为我们推断类型，这时候就会回退到最宽泛的类型：any 。
这倒不是最糟糕的事情，毕竟回退到 any就跟我们写 JavaScript 没啥一样了。

但是，经常使用 any 有违背我们使用 TypeScript 的目的。你程序使用的类型越多，你在验证和工具上得到的帮助就会越多，
这也意味着写代码的时候会遇到更少的 bug。启用 noImplicitAny配置项后，当类型被隐式推断为 any 时，会抛出一个错误。


### strictNullChecks

默认情况下，像 null 和 undefined 这样的值可以赋值给其他的类型。这可以让我们更方便的写一些代码。
但是忘记处理 null 和 undefined 也导致了不少的 bug，甚至有些人会称呼它为价值百万的错误！ 


strictNullChecks 选项会让我们更明确的处理 null 和 undefined，也会让我们免于忧虑是否忘记处理 null 和 undefined 。