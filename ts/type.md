### 原始类型：string, number 和 boolean

string：表示字符串，比如 "hello,world"

number 表示数字，比如 42（没有 int 或者 float，所有的数字类型都是 number）

boolean 表示布尔值，两个值 true 和 false

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

obj.foo();
obj();
obj.bar = 100;
obj = "hello";
const n: number = obj;
```

#### noImplicitAny

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

let myName: string;
```

### 函数（Function）

函数是 JavaScript 传递数据的主要方法。TypeScript 允许你指定函数的输入值和输出值的类型。

#### 参数类型注解

当你声明一个函数的时候，你可以在每个参数后面添加一个类型注解，声明函数可以接受什么类型的参数。

```ts
function greet(name: string) {
  console.log("Hello, " + name.toUpperCase() + "!!");
}
```

当参数有了类型注解的时候，TypeScript 便会检查函数的实参：

```ts
greet(42); // 类型“number”的参数不能赋给类型“string”的参数。ts(2345)
```

即便你对参数没有做类型注解，TypeScript 依然会检查传入参数的数量是否正确

```ts
function greet(name, age) {
  console.log(`Hello, ${name.toUpperCase()}!!`);
}
greet(42); // 未提供 "age" 的自变量。
```

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
const names = ["Alice", "Bob", "Eve"];

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
function printCoord(pt: { x: number; y: number }): void;

// The parameter's type annotation is an object type
function printCoord(pt: { x: number; y: number }) {
  console.log("The coordinate's x value is " + pt.x);
  console.log("The coordinate's y value is " + pt.y);
}

printCoord({ x: 3, y: 7 });
```

我们给参数添加了一个类型，该类型有两个属性, x 和 y，两个都是 number 类型。你可以使用 , 或者 ; 分开属性，最后一个属性的分隔符加不加都行。

每个属性对应的类型是可选的，如果你不指定，默认使用 any 类型。

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

在 JavaScript 中，如果你获取一个不存在的属性，你会得到一个 undefined 而不是一个运行时错误。
因此，当你获取一个可选属性时，你需要在使用它前，先检查一下是否是 undefined。

```ts
function printName(obj: { first: string; last?: string }) {
  console.log(obj.last.toUpperCase());
  // (property) last?: string | undefined
  // 对象可能为“未定义”。ts(2532)

  if (obj.last !== undefined) {
    // OK
    console.log(obj.last.toUpperCase());
  }

  // better
  console.log(obj.last?.toUpperCase());
}
```

### 联合类型

ts 类型系统允许你使用一系列的操作符，基于已经存在的类型构建新的类型。
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
```

另一个例子，使用 Array.isArray

```ts
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

### 类型别名

一个类型会被使用多次，此时我们更希望通过一个单独的名字来引用它。这就是类型别名。

所谓类型别名，顾名思义，一个可以指代任意类型的名字。类型别名的语法是：

```ts
type Point = {
  x: number;
  y: number;
};

// Exactly the same as the earlier example
function printCoord(pt: Point) {
  console.log("The coordinate's x value is " + pt.x);
  console.log("The coordinate's y value is " + pt.y);
}

printCoord({ x: 100, y: 100 });
```

用类型别名给任意类型一个名字，举个例子，命名一个联合类型：

```ts
type ID = number | string;
```

注意别名是唯一的别名，你不能使用类型别名创建同一个类型的不同版本。当你使用类型别名的时候，它就跟你编写的类型是一样的。换句话说，代码看起来可能不合法，但对 TypeScript 依然是合法的，因为两个类型都是同一个类型的别名:

```ts
type UserInputSanitizedString = string;

function sanitizeInput(str: string): UserInputSanitizedString {
  return sanitize(str);
}

// Create a sanitized input
let userInput = sanitizeInput(getInput());

// Can still be re-assigned with a string though
userInput = "new input";
```

### 接口

接口声明是命名对象类型的另一种方式：

```ts
interface Point {
  x: number;
  y: number;
}

function printCoord(pt: Point) {
  console.log("The coordinate's x value is " + pt.x);
  console.log("The coordinate's y value is " + pt.y);
}

printCoord({ x: 100, y: 100 });
```

#### 类型别名和接口的不同

类型别名和接口非常相似，大部分时候，你可以任意选择使用。接口的几乎所有特性都可以在 type 中使用，
两者最关键的差别在于类型别名本身无法添加新的属性，而接口是可以扩展的。

```ts
// Interface
// 通过继承扩展类型
interface Animal {
  name: string;
}

interface Bear extends Animal {
  honey: boolean;
}

const bear = getBear();
bear.name;
bear.honey;

// Type
// 通过交集扩展类型
type Animal = {
  name: string;
};

type Bear = Animal & {
  honey: boolean;
};

const bear = getBear();
bear.name;
bear.honey;
```

```ts
// Interface
// 对一个已经存在的接口添加新的字段
interface Window {
  title: string;
}

interface Window {
  ts: TypeScriptAPI;
}

const src = 'const a = "Hello World"';
window.ts.transpileModule(src, {});

// Type
// 创建后不能被改变
type Window = {
  title: string;
};

type Window = {
  ts: TypeScriptAPI;
};

// Error: Duplicate identifier 'Window'.
```

```ts
interface Mammal {
  name: string;
}

function echoMammal(m: Mammal) {
  console.log(m.name);
}

// (property) Mammal.name: string
// 不能将类型“number”分配给类型“string”。ts(2322)
echoMammal({ name: 12343 });

type Lizard = {
  name: string;
};

function echoLizard(l: Lizard) {
  console.log(l.name);
}

// (property) name: string
// 不能将类型“number”分配给类型“string”。ts(2322)
echoLizard({ name: 12345 });
```

?> 类型别名也许不会实现声明合并，但是接口可以  
接口可能只会被用于声明对象的形状，不能重命名原始类型  
使用接口类型出错时，接口的名字则会始终出现在错误信息中

### 类型断言

有的时候，你知道一个值的类型，但 TypeScript 不知道。

举个例子，如果你使用 document.getElementById，TypeScript 仅仅知道它会返回一个 HTMLElement，但是你却知道，你要获取的是一个 HTMLCanvasElement。

```ts
const myCanvas = document.getElementById("main_canvas") as HTMLCanvasElement;

// 等价

const myCanvas = <HTMLCanvasElement>document.getElementById("main_canvas");
```

就像类型注解一样，类型断言也会被编译器移除，并且不会影响任何运行时的行为。即使类型断言是错误的，也不会有异常或者 null 产生。

### 字面量类型

除了常见的类型 string 和 number ，我们也可以将类型声明为更具体的数字或者字符串。

字面量类型本身并没有什么太大用：

```ts
let x: "hello" = "hello";
x = "hello";
x = "howdy"; //不能将类型“"howdy"”分配给类型“"hello"”。ts(2322)
```

如果结合联合类型，就显得有用多了。举个例子，当函数只能传入一些固定的字符串时：

```ts
function printText(s: string, alignment: "left" | "right" | "center") {
  // ...
}
printText("Hello, world", "left");
printText("G'day, mate", "centre"); // 类型“"centre"”的参数不能赋给类型“"left" | "right" | "center"”的参数。ts(2345)
```

数字字面量

```ts
function compare(a: string, b: string): -1 | 0 | 1 {
  return a === b ? 0 : a > b ? 1 : -1;
}
```

非字面量类型联合

```ts
interface Options {
  width: number;
}
function configure(x: Options | "auto") {
  // ...
}
configure({ width: 100 });
configure("auto");
configure("automatic"); // 类型“"automatic"”的参数不能赋给类型“Options | "auto"”的参数。
```

#### 字面量推断

```ts
declare function handleRequest(url: string, method: "GET" | "POST"): void;

const req = { url: "https://example.com", method: "GET" };
handleRequest(req.url, req.method); // 类型“string”的参数不能赋给类型“"GET" | "POST"”的参数。ts(2345)
```

req.method 被推断为 string ，而不是 "GET"，因为在创建 req 和 调用 handleRequest 函数之间，可能还有其他的代码，或许会将 req.method 赋值一个新字符串比如 "Guess" 。所以 TypeScript 就报错了。

解决

1、添加一个类型断言改变推断结果：

```ts
declare function handleRequest(url: string, method: "GET" | "POST"): void;
const req = { url: "https://example.com", method: "GET" };
handleRequest(req.url, req.method as "GET");
```

2、使用 as const 把整个对象转为一个类型字面量

```ts
declare function handleRequest(url: string, method: "GET" | "POST"): void;
const req = { url: "https://example.com", method: "GET" } as const;
handleRequest(req.url, req.method);
```

as const 效果跟 const 类似，但是对类型系统而言，它可以确保所有的属性都被赋予一个字面量类型，而不是一个更通用的类型比如 string 或者 number 。

### null 和 undefined

JavaScript 有两个原始类型的值，用于表示空缺或者未初始化，他们分别是 null 和 undefined 。

#### strictNullChecks 关闭

当 strictNullChecks 选项关闭的时候，如果一个值可能是 null 或者 undefined，它依然可以被正确的访问，或者被赋值给任意类型的属性。

#### strictNullChecks 打开

如果一个值可能是 null 或者 undefined，你需要在用它的方法或者属性之前，先检查这些值，就像用可选的属性之前，先检查一下是否是 undefined ，我们也可以使用类型收窄检查值是否是 null

```ts
function doSomething(x: string | null) {
  if (x === null) {
    // do nothing
  } else {
    console.log("Hello, " + x.toUpperCase());
  }
}
```

### 非空断言操作符（后缀 !）

ypeScript 提供了一个特殊的语法，可以在不做任何检查的情况下，从类型中移除 null 和 undefined，这就是在任意表达式后面写上 ! ，
这是一个有效的类型断言，表示它的值不可能是 null 或者 undefined：

```ts
function liveDangerously(x?: number | null) {
  // No error
  console.log(x!.toFixed());
}
```

重要的事情说一遍，只有当你明确的知道这个值不可能是 null 或者 undefined 时才使用 ! 。

### 枚举

枚举是 TypeScript 添加的新特性，用于描述一个值可能是多个常量中的一个。这并不是一个类型层面的增量，而是会添加到语言和运行时。

### 不常见的原始类型

#### bigInt

ES2020 引入原始类型 BigInt，用于表示非常大的整数：

```ts
// Creating a bigint via the BigInt function
const oneHundred: bigint = BigInt(100);
```

#### symbol

过函数 Symbol()，我们可以创建一个全局唯一的引用：

```ts
const firstName = Symbol("name");
const secondName = Symbol("name");

if (firstName === secondName) {
  // This condition will always return 'false' since the types 'typeof firstName' and 'typeof secondName' have no overlap.
  // Can't ever happen
}
```
