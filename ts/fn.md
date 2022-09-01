### 函数

函数也是值 (values)，而且像其他值一样，TypeScript 有很多种方式用来描述，函数可以以怎样的方式被调用。

### 函数类型表达式

1、最简单描述一个函数的方式是使用函数类型表达式

```ts
function greeter(fn: (a: string) => void) {
  fn("Hello, World");
}

function printToConsole(s: string) {
  console.log(s);
}

greeter(printToConsole);
```

语法 (a: string) => void 表示一个函数有一个名为 a 类型是字符串的参数，这个函数并没有返回任何值。

一个函数参数的类型并没有明确给出，它会被隐式设置为 any

```ts
// (parameter) string: any
function greeter(fn: (string) => void) {
  fn("Hello, World");
}
```

函数类型描述 (string) => void，表示的其实是一个函数有一个类型是 any，名为 string 的参数。

2、使用类型别名定义一个函数类型

```ts
type GreetFunction = (a: string) => void;
function greeter(fn: GreetFunction) {
  // ...
}
```

### 调用签名

在 JavaScript 中，函数除了可以被调用，自己也是可以有属性值的。  
描述一个带有属性的函数，我们可以在一个对象类型中写一个调用签名。

```ts
type DescribableFunction = {
  description: string;
  (someArg: number): boolean;
};

function doSomething(fn: DescribableFunction) {
  console.log(fn.description + " returned " + fn(6));
}

function MoreThanFive(someArg: number) {
  return someArg > 5 ? true : false;
}

MoreThanFive.description = "函数是否大于 5";

doSomething(MoreThanFive);
```

### 构造签名

JavaScript 函数也可以使用 new 操作符调用，当被调用的时候，TypeScript 会认为这是一个构造函数，因为他们会产生一个新对象。你可以写一个构造签名，方法是在调用签名前面加一个 new 关键词：

```ts
type SomeConstructor = {
  new (s: string): SomeObject;
};

function fn(ctor: SomeConstructor) {
  return new ctor("hello");
}
```

一些对象，比如 Date 对象，可以直接调用，也可以使用 new 操作符调用，而你可以将调用签名和构造签名合并在一起：

```ts
interface CallOrConstruct {
  new (s: string): Date;
  (n?: number): number;
}
```

### 泛型函数

函数的输出类型依赖函数的输入类型，或者两个输入的类型以某种形式相互关联

泛型就是被用来描述两个值之间的对应关系。我们需要在函数签名里声明一个类型参数：

考虑这样一个函数，它返回数组的第一个元素

```ts
function firstElement<Type>(arr: Type[]): Type | undefined {
  return arr[0];
}
```

通过给函数添加一个类型参数 Type，并且在两个地方使用它，在函数的输入(即数组)和函数的输出(即返回值)之间创建了一个关联。当调用它，一个更具体的类型就会被判断出来：

```ts
// const s: string | undefined
const s = firstElement(["a", "b", "c"]);
// const n: number | undefined
const n = firstElement([1, 2, 3]);
// const u: undefined
const u = firstElement([]);
```

#### 推断

上面例子，没有明确指定 Type 的类型，类型是被 TypeScript 自动推断出来的

```ts
function map<Input, Output>(
  arr: Input[],
  func: (arg: Input) => Output
): Output[] {
  return arr.map(func);
}

// Parameter 'n' is of type 'string'
// 'parsed' is of type 'number[]'
const parsed = map(["1", "2", "3"], (n) => parseInt(n));
```

TypeScript 既可以推断出 Input 的类型 （从传入的 string 数组），又可以根据函数表达式的返回值推断出 Output 的类型。

#### 约束

有的时候，我们想关联两个值，但只能操作值的一些固定字段，这种情况，我们可以使用**约束**对类型参数进行限制。

写一个函数，函数返回两个值中更长的那个。为此，我们需要保证传入的值有一个 number 类型的 length 属性。使用 extends 语法来约束函数参数

```ts
function longest<Type extends { length: number }>(a: Type, b: Type) {
  if (a.length >= b.length) {
    return a;
  } else {
    return b;
  }
}

// const longerArray: number[]
const longerArray = longest([1, 2], [1, 2, 3]);

// const longerString: "alice" | "bob"
const longerString = longest("alice", "bob");

// const notOK: { length: number };
// 类型“number”的参数不能赋给类型“{ length: number; }”的参数
const notOK = longest(10, 100);
```

TypeScript 会推断 longest 的返回类型，所以返回值的类型推断在泛型函数里也是适用的。

正是因为我们对 Type 做了 { length: number } 限制，我们才可以被允许获取 a b 参数的 .length 属性。没有这个类型约束，我们甚至不能获取这些属性，因为这些值也许是其他类型，并没有 length 属性。

基于传入的参数，longerArray 和 longerString 中的类型都被推断出来了。记住，所谓泛型就是用一个相同类型来关联两个或者更多的值。

#### 泛型约束实战

```ts
function minimumLength<Type extends { length: number }>(
  obj: Type,
  minimum: number
): Type {
  if (obj.length >= minimum) {
    return obj;
  } else {
    // 不能将类型“{ length: number; }”分配给类型“Type”。
    // "{ length: number; }" 可赋给 "Type" 类型的约束，但可以使用约束 "{ length: number; }" 的其他子类型实例化 "Type"。ts(2322)
    return { length: minimum };
  }
}
```

Type 被 { length: number} 约束，函数返回 Type 或者一个符合约束的值。

函数理应返回与传入参数相同类型的对象，而不仅仅是符合约束的对象。我们可以写出这样一段反例：

```ts
// 'arr' gets value { length: 6 }
const arr = minimumLength([1, 2, 3], 6);
// and crashes here because arrays have
// a 'slice' method, but not the returned object!
console.log(arr.slice(0));
```

#### 声明类型参数

TypeScript 通常能自动推断泛型调用中传入的类型参数，但也并不能总是推断出。举个例子，有这样一个合并两个数组的函数：

```ts
function combine<Type>(arr1: Type[], arr2: Type[]): Type[] {
  return arr1.concat(arr2);
}
```

```ts
// ["hello"] 不能将类型“string”分配给类型“number”。
// const arr: number[]
const arr = combine([1, 2, 3], ["hello"]);
```

而如果你执意要这样做，你可以手动指定 Type：

```ts
// const arr: (string | number)[]
const arr = combine<string | number>([1, 2, 3], ["hello"]);
```

#### 写一个好的泛型函数的一些建议

如果你使用了太多的类型参数，或者使用了一些并不需要的约束，都可能会导致不正确的类型推断。

##### 使用更少的类型参数

```ts
function filter1<Type>(arr: Type[], func: (arg: Type) => boolean): Type[] {
  return arr.filter(func);
}

function filter2<Type, Func extends (arg: Type) => boolean>(
  arr: Type[],
  func: Func
): Type[] {
  return arr.filter(func);
}
```

我们创建了一个并没有关联两个值的类型参数 Func，这是一个危险信号，因为它意味着调用者不得不毫无理由的手动指定一个额外的类型参数。Func 什么也没做，却导致函数更难阅读和推断。

##### 类型参数应该出现两次

类型参数是用来关联多个值之间的类型。如果一个类型参数只在函数签名里出现了一次，那它就没有跟任何东西产生关联。

```ts
function greet<Str extends string>(s: Str) {
  console.log("Hello, " + s);
}

greet("world");
```

```ts
function greet(s: string) {
  console.log("Hello, " + s);
}
```

### 可选参数

js 函数经常会被传入非固定数量的参数

```ts
function f(n: number) {
  console.log(n.toFixed()); // 0 arguments
  console.log(n.toFixed(3)); // 1 argument
}
```

我们可以使用 ? 表示这个参数是可选的：

```ts
function f(x?: number) {
  // ...
}
f(); // OK
f(10); // OK
```

尽管这个参数被声明为 number 类型，x 实际上的类型为 number | undefiend，这是因为在 JavaScript 中未指定的函数参数就会被赋值 undefined。

你当然也可以提供有一个参数默认值：

```ts
function f(x = 10) {
  // ...
}
```

现在在 f 函数体内，x 的类型为 number，因为任何 undefined 参数都会被替换为 10。注意当一个参数是可选的，调用的时候还是可以传入 undefined：

```ts
declare function f(x?: number): void;
// cut
// All OK
f();
f(10);
f(undefined);
```

#### 回调中的可选参数

```ts
function myForEach(arr: any[], callback: (arg: any, index?: number) => void) {
  for (let i = 0; i < arr.length; i++) {
    callback(arr[i], i);
  }
}

myForEach([1, 2, 3], (a) => console.log(a));
myForEach([1, 2, 3], (a, i) => console.log(a, i));
```

```ts
function myForEach(arr: any[], callback: (arg: any, index: number) => void) {
  for (let i = 0; i < arr.length; i++) {
    callback(arr[i], i);
  }
}

myForEach([1, 2, 3], (a) => {
  console.log(a);
});
```

### 函数重载

函数在调用的时候可以传入不同数量和类型的参数。

你可以写一个函数，返回一个日期类型 Date，这个函数接收一个时间戳（一个参数）或者一个 月/日/年 的格式 (三个参数)。

可以通过写重载签名说明一个函数的不同调用方法。 我们需要写一些函数签名 (通常两个或者更多)，然后再写函数体的内容：

```ts
function makeDate(timestamp: number): Date;
function makeDate(m: number, d: number, y: number): Date;
function makeDate(mOrTimestamp: number, d?: number, y?: number): Date {
  if (d !== undefined && y !== undefined) {
    return new Date(y, mOrTimestamp, d);
  } else {
    return new Date(mOrTimestamp);
  }
}
const d1 = makeDate(12345678);
const d2 = makeDate(5, 5, 5);
const d3 = makeDate(1, 3); //没有需要 2 参数的重载，但存在需要 1 或 3 参数的重载。ts(2575)
```

写了两个函数重载，一个接受一个参数，另外一个接受三个参数。前面两个函数签名被称为重载签名。然后写了一个兼容签名的函数实现，我们称之为实现签名，但这个签名不能被直接调用。尽管我们在函数声明中，在一个必须参数后，声明了两个可选参数，它依然不能被传入两个参数进行调用。

#### 重载签名和实现签名

```ts
function fn(x: string): void;
function fn() {
  // ...
}
fn(); //应有 1 个参数，但获得 0 个。ts(2554)
```

实现签名对外界来说是不可见的。当写一个重载函数的时候，你应该总是需要来两个或者更多的签名在实现签名之上。

#### 写一个好的函数重载的一些建议

```ts
function len(s: string): number;
function len(arr: any[]): number;
function len(x: any) {
  return x.length;
}

len(""); // OK
len([0]); // OK
len(Math.random() > 0.5 ? "hello" : [0]);
// 没有与此调用匹配的重载。
//   第 1 个重载(共 2 个)，“(s: string): number”，出现以下错误。
//     类型“number[] | "hello"”的参数不能赋给类型“string”的参数。
//       不能将类型“number[]”分配给类型“string”。
//   第 2 个重载(共 2 个)，“(arr: any[]): number”，出现以下错误。
//     类型“number[] | "hello"”的参数不能赋给类型“any[]”的参数。
//       不能将类型“string”分配给类型“any[]”。ts(2769)
// 针对此实现的调用已成功，但重载的实现签名在外部不可见。
```

因为两个函数重载都有相同的参数数量和相同的返回类型，我们可以写一个无重载版本的函数替代：

```ts
function len(x: any[] | string) {
  return x.length;
}
```

尽可能的使用联合类型替代重载

#### 在函数中声明 this

```ts
const user = {
  id: 123,

  admin: false,
  becomeAdmin: function () {
    this.admin = true;
  },
};

user.becomeAdmin();
```

TypeScript 能够理解函数 user.becomeAdmin 中的 this 指向的是外层的对象 user，这已经可以应付很多情况了，但还是有一些情况需要你明确的告诉 TypeScript this 到底代表的是什么。

在 JavaScript 中，this 是保留字，所以不能当做参数使用。但 TypeScript 可以允许你在函数体内声明 this 的类型。

```ts
interface DB {
  filterUsers(filter: (this: User) => boolean): User[];
}

const db = getDB();
const admins = db.filterUsers(function (this: User) {
  return this.admin;
});
```

这个写法有点类似于回调风格的 API。注意你需要使用 function 的形式而不能使用箭头函数：

```ts
interface DB {
  filterUsers(filter: (this: User) => boolean): User[];
}

const db = getDB();
const admins = db.filterUsers(() => this.admin);
// The containing arrow function captures the global value of 'this'.
// Element implicitly has an 'any' type because type 'typeof globalThis' has no index signature.
```

### 其他类型

这里介绍一些也会经常出现的类型。像其他的类型一样，你也可以在任何地方使用它们，但它们经常与函数搭配使用。

#### void

void 表示一个函数并不会返回任何值，当函数并没有任何返回值，或者返回不了明确的值的时候，就应该用这种类型。

```ts
// function noop(): void
function noop() {
  return;
}
```

在 JavaScript 中，一个函数并不会返回任何值，会隐式返回 undefined，但是 void 和 undefined 在 TypeScript 中并不一样。

#### object

这个特殊的类型 object 可以表示任何不是原始类型的值 (string、number、bigint、boolean、symbol、null、undefined)。object 不同于空对象类型 { }，也不同于全局类型 Object。很有可能你也用不到 Object。

注意在 JavaScript 中，函数就是对象，他们可以有属性，在他们的原型链上有 Object.prototype，并且 instanceof Object。你可以对函数使用 Object.keys 等等。由于这些原因，在 TypeScript 中，函数也被认为是 object。

#### unknown

unknown 类型可以表示任何值。有点类似于 any，但是更安全，因为对 unknown 类型的值做任何事情都是不合法的：

```ts
function f1(a: any) {
  a.b(); // OK
}
function f2(a: unknown) {
  a.b(); // 对象的类型为 "unknown"。ts(2571)
}
```

有的时候用来描述函数类型，还是蛮有用的。你可以描述一个函数可以接受传入任何值，但是在函数体内又不用到 any 类型的值。

你可以描述一个函数返回一个不知道什么类型的值，比如：

```ts
function safeParse(s: string): unknown {
  return JSON.parse(s);
}

const obj = safeParse(someRandomString);
```

#### never

never 类型表示一个值不会再被观察到。 (穷尽类型)

当 TypeScript 确定在联合类型中已经没有可能是其中的类型的时候，never 类型也会出现：

```ts
function fn(x: string | number) {
  if (typeof x === "string") {
    // do something
  } else if (typeof x === "number") {
    // do something else
  } else {
    x; // (parameter) x: never
  }
}
```

#### Function

在 JavaScript，全局类型 Function 描述了 bind、call、apply 等属性，以及其他所有的函数值。

它也有一个特殊的性质，就是 Function 类型的值总是可以被调用，结果会返回 any 类型：

```ts
function doSomething(f: Function) {
  f(1, 2, 3);
}
```

这是一个无类型函数调用，这种调用最好被避免，因为它返回的是一个不安全的 any 类型。

如果你准备接受一个黑盒的函数，但是又不打算调用它，() => void 会更安全一些。

### 剩余参数

#### parameters 与 arguments

arguments 和 parameters 都可以表示函数的参数，由于本节内容做了具体的区分，所以我们定义 parameters 表示我们定义函数时设置的名字即形参，arguments 表示我们实际传入函数的参数即实参。

#### 剩余参数 (Rest Parameters)

除了用可选参数、重载能让函数接收不同数量的函数参数，我们也可以通过使用剩余参数语法，定义一个可以传入数量不受限制的函数参数的函数：

剩余参数必须放在所有参数的最后面，并使用 ... 语法：

```ts
function multiply(n: number, ...m: number[]) {
  return m.map((x) => n * x);
}
// 'a' gets value [10, 20, 30, 40]
const a = multiply(10, 1, 2, 3, 4);
```

在 TypeScript 中，剩余参数的类型会被隐式设置为 any[] 而不是 any，如果你要设置具体的类型，必须是 Array<T> 或者 T[]的形式，再或者就是元组类型

#### 剩余参数（Rest Arguments）

可以借助一个使用 ... 语法的数组，为函数提供不定数量的实参。举个例子，数组的 push 方法就可以接受任何数量的实参：

```ts
const arr1 = [1, 2, 3];
const arr2 = [4, 5, 6];
arr1.push(...arr2);
```

### 解构参数

使用参数解构方便的将作为参数提供的对象解构为函数体内一个或者多个局部变量;

```ts
function sum({ a, b, c }) {
  console.log(a + b + c);
}
sum({ a: 10, b: 3, c: 9 });
```

在解构语法后，要写上对象的类型注解：

```ts
function sum({ a, b, c }: { a: number; b: number; c: number }) {
  console.log(a + b + c);
}
// or
type ABC = { a: number; b: number; c: number };
function sum({ a, b, c }: ABC) {
  console.log(a + b + c);
}
```

### 函数的可赋值性

#### 返回 void

函数有一个 void 返回类型，会产生一些意料之外，情理之中的行为。

当基于上下文的类型推导出返回类型为 void 的时候，并不会强制函数一定不能返回内容。换句话说，如果这样一个返回 void 类型的函数类型 (type vf = () => void)， 当被应用的时候，也是可以返回任何值的，但返回的值会被忽略掉。

```ts
type voidFunc = () => void;

const f1: voidFunc = () => {
  return true;
};

const f2: voidFunc = () => true;

const f3: voidFunc = function () {
  return true;
};

// const v1: void
const v1 = f1();

// const v2: void
const v2 = f2();

// const v3: void
const v3 = f3();
```

正是因为这个特性的存在，所以接下来的代码才会是有效的：

```ts
const src = [1, 2, 3];
const dst = [0];

src.forEach((el) => dst.push(el));
```

尽管 Array.prototype.push 返回一个数字，并且 Array.prototype.forEach 方法期待一个返回 void 类型的函数，但这段代码依然没有报错。就是因为基于上下文推导，推导出 forEach 函数返回类型为 void，正是因为不强制函数一定不能返回内容，所以上面这种 return dst.push(el) 的写法才不会报错。
