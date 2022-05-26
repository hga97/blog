### 类型别名

一个类型会被使用多次，此时我们更希望通过一个单独的名字来引用它。

这就是类型别名（type alias）。所谓类型别名，顾名思义，一个可以指代任意类型的名字。类型别名的语法是：

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

### 接口

接口声明（interface declaration）是命名对象类型的另一种方式：

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
  name: string
}

interface Bear extends Animal {
  honey: boolean
}

const bear = getBear() 
bear.name
bear.honey
        
// Type
// 通过交集扩展类型
type Animal = {
  name: string
}

type Bear = Animal & { 
  honey: boolean 
}

const bear = getBear();
bear.name;
bear.honey;

```

```ts
// Interface
// 对一个已经存在的接口添加新的字段
interface Window {
  title: string
}

interface Window {
  ts: TypeScriptAPI
}

const src = 'const a = "Hello World"';
window.ts.transpileModule(src, {});
        
// Type
// 创建后不能被改变
type Window = {
  title: string
}

type Window = {
  ts: TypeScriptAPI
}

// Error: Duplicate identifier 'Window'.

```

?> 类型别名也许不会实现声明合并，但是接口可以  
接口可能只会被用于声明对象的形状，不能重命名原始类型


### 类型断言

有的时候，你知道一个值的类型，但 TypeScript 不知道。

举个例子，如果你使用 document.getElementById，TypeScript 仅仅知道它会返回一个 HTMLElement，但是你却知道，你要获取的是一个 HTMLCanvasElement。

```ts
const myCanvas = document.getElementById("main_canvas") as HTMLCanvasElement;

```

就像类型注解一样，类型断言也会被编译器移除，并且不会影响任何运行时的行为。

### 字面量类型

除了常见的类型 string 和 number ，我们也可以将类型声明为更具体的数字或者字符串。

```ts
function printText(s: string, alignment: "left" | "right" | "center") {
  // ...
}
printText("Hello, world", "left");
printText("G'day, mate", "centre");
// 类型“"centre"”的参数不能赋给类型“"left" | "right" | "center"”的参数。ts(2345)

```

### null和undefined

JavaScript 有两个原始类型的值，用于表示空缺或者未初始化，他们分别是 null 和 undefined 。

#### strictNullChecks 关闭

当 strictNullChecks 选项关闭的时候，如果一个值可能是 null 或者 undefined，它依然可以被正确的访问，或者被赋值给任意类型的属性。

#### strictNullChecks 打开

如果一个值可能是 null 或者 undefined，你需要在用它的方法或者属性之前，先检查这些值，就像用可选的属性之前，先检查一下 是否是 undefined ，我们也可以使用类型收窄（narrowing）检查值是否是 null


### 非空断言操作符（后缀 !）

ypeScript 提供了一个特殊的语法，可以在不做任何检查的情况下，从类型中移除 null 和 undefined，这就是在任意表达式后面写上 ! ，
这是一个有效的类型断言，表示它的值不可能是 null 或者 undefined：


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
