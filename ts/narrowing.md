### 类型收窄

TypeScript 的类型检查器会考虑到这些类型保护和赋值语句，而这个将类型推导为更精确类型的过程，我们称之为收窄。

```ts
function padLeft(padding: number | string, input: string) {
  if (typeof padding === "number") {
    return new Array(padding + 1).join(" ") + input; // (parameter) padding: number
  }
  return padding + input; // (parameter) padding: string
}
```

### typeof 类型保护

JavaScript 本身就提供了 typeof 操作符，可以返回运行时一个值的基本类型信息。
string、number、boolean、object、function

```ts
function printAll(strs: string | string[] | null) {
  if (typeof strs === "object") {
    for (const s of strs) {
      // Object is possibly 'null'.
      console.log(s);
    }
  } else if (typeof strs === "string") {
    console.log(strs);
  } else {
    // do nothing
  }
}
```

TypeScript 会让我们知道 strs 被收窄为 strings[] | null ，而不仅仅是 string[]。

### 真值收窄

在 JavaScript 中，我们可以在条件语句中使用任何表达式，比如 && 、||、! 等，举个例子，
像 if 语句就不需要条件的结果总是 boolean 类型

```ts
function getUsersOnlineMessage(numUsersOnline: number) {
  if (numUsersOnline) {
    return `There are ${numUsersOnline} online now!`;
  }
  return "Nobody's here. :(";
}
```

这是因为 JavaScript 会做隐式类型转换，像 0 、NaN、""、null、 undefined 这些值都会被转为 false，其他的值则会被转为 true。

当然你也可以使用 Boolean 函数强制转为 boolean 值，或者使用更加简短的!!：

```ts
Boolean("hello");
!!"world";
```

### 等值收窄

```ts
function printAll(strs: string | string[] | null) {
  if (strs ！== null) {
    if (typeof strs === "object") {
      // (parameter) strs: string[]
      for (const s of strs) {
        console.log(s);
      }
    } else if (typeof strs === "string") {
      // (parameter) strs: string
      console.log(strs);
    }
  }
}
```

### in 操作符收窄

JavaScript 中有一个 in 操作符可以判断一个对象是否有对应的属性名。TypeScript 也可以通过这个收窄类型。

```ts
type Fish = { swim: () => void };
type Bird = { fly: () => void };

function move(animal: Fish | Bird) {
  if ("swim" in animal) {
    return animal.swim();
    // (parameter) animal: Fish
  }

  return animal.fly();
  // (parameter) animal: Bird
}
```

通过 "swim" in animal ，我们可以准确的进行类型收窄。

```ts
type Fish = { swim: () => void };
type Bird = { fly: () => void };
type Human = { swim?: () => void; fly?: () => void };

function move(animal: Fish | Bird | Human) {
  if ("swim" in animal) {
    animal; // (parameter) animal: Fish | Human
  } else {
    animal; // (parameter) animal: Bird | Human
  }
}
```

### instanceof 收窄

instanceof 也是一种类型保护，TypeScript 也可以通过识别 instanceof 正确的类型收窄：

instanceof 运算符用来测试一个对象在其原型链中是否存在一个构造函数的 prototype 属性。

```ts
function logValue(x: Date | string) {
  if (x instanceof Date) {
    // (parameter) x: Date
    console.log(x.toUTCString());
  } else {
    // (parameter) x: string
    console.log(x.toUpperCase());
  }
}
```

### 赋值语句

TypeScript 可以根据赋值语句的右值，正确的收窄左值。

```ts
let x = Math.random() < 0.5 ? 10 : "hello world"; //let x: string | number
x = 1;
x = "goodbye";
x = true; // 不能将类型“boolean”分配给类型“string | number”。ts(2322)
```

### 控制流程分析

```ts
function padLeft(padding: number | string, input: string) {
  if (typeof padding === "number") {
    return new Array(padding + 1).join(" ") + input;
  }
  return padding + input;
}
```

在第一个 if 语句里，因为有 return 语句，TypeScript 就能通过代码分析，判断出在剩余的部分 return padding + input ，如果 padding 是 number 类型，是无法达到这里的，所以在剩余的部分，就会将 number 类型从 number | string 类型中删除掉。

这种基于可达性的代码分析就叫做控制流分析。在遇到类型保护和赋值语句的时候，TypeScript 就是使用这样的方式收窄类型。而使用这种方式，一个变量可以被观察到变为不同的类型：

```ts
function example() {
  let x: string | number | boolean;

  x = Math.random() < 0.5;
  console.log(x); // let x: boolean

  if (Math.random() < 0.5) {
    x = "hello";
    console.log(x); // let x: string
  } else {
    x = 100;
    console.log(x); // let x:number
  }
  return x; // let x: string | number
}
```

### 类型判断式

搞不懂

### 可辨别联合

```ts
interface Shape {
  kind: "circle" | "square";
  radius?: number;
  sideLength?: number;
}
```

获取面积的 getArea 函数

```ts
function getArea(shape: Shape) {
  return Math.PI * shape.radius ** 2; // 圆的面积公式 S=πr²
  // (property) Shape.radius?: number | undefined 对象可能为“未定义”
}
```

加上 kind 判断，依然报错

```ts
function getArea(shape: Shape) {
  if (shape.kind === "circle") {
    return Math.PI * shape.radius ** 2;
    // (property) Shape.radius?: number | undefined 对象可能为“未定义”
  }
}
```

Shape 的问题在于类型检查器并没有方法根据 kind 属性，判断 radius 和 sideLength 属性是否存在

```ts
interface Circle {
  kind: "circle";
  radius: number;
}

interface Square {
  kind: "square";
  sideLength: number;
}

type Shape = Circle | Square;

function getArea(shape: Shape) {
  return Math.PI * shape.radius ** 2;
  // 类型“Shape”上不存在属性“radius”。
  // 类型“Square”上不存在属性“radius”。ts(2339)
}
```

当联合类型中的每个类型，都包含了一个共同的字面量类型的属性，TypeScript 就会认为这是一个可辨别联合，然后可以将具体成员的类型进行收窄。

```ts
function getArea(shape: Shape) {
  switch (shape.kind) {
    case "circle":
      // (parameter) shape: Circle
      return Math.PI * shape.radius ** 2;
    case "square":
      // (parameter) shape: Square
      return shape.sideLength ** 2;
  }
}
```

### never 类型

当进行收窄的时候，如果你把所有可能的类型都穷尽了，TypeScript 会使用一个 never 类型来表示一个不可能存在的状态。

### 穷尽检查

never 类型可以赋值给任何类型，然而，没有类型可以赋值给 never （除了 never 自身）。这就意味着你可以在 switch 语句中使用 never 来做一个穷尽检查。

```ts
function getArea(shape: Shape) {
  switch (shape.kind) {
    case "circle":
      return Math.PI * shape.radius ** 2;
    case "square":
      return shape.sideLength ** 2;
    default:
      const _exhaustiveCheck: never = shape;
      return _exhaustiveCheck;
  }
}
```

当我们给 Shape 类型添加一个新成员，却没有做对应处理的时候，就会导致一个 TypeScript 错误：

```ts
interface Triangle {
  kind: "triangle";
  sideLength: number;
}

type Shape = Circle | Square | Triangle;

function getArea(shape: Shape) {
  switch (shape.kind) {
    case "circle":
      return Math.PI * shape.radius ** 2;
    case "square":
      return shape.sideLength ** 2;
    default: //不能将类型“Triangle”分配给类型“never”。ts(2322)
      const _exhaustiveCheck: never = shape;
      return _exhaustiveCheck;
  }
}
```

因为 TypeScript 的收窄特性，执行到 default 的时候，类型被收窄为 Triangle，但因为任何类型都不能赋值给 never 类型，这就会产生一个编译错误。通过这种方式，你就可以确保 getArea 函数总是穷尽了所有 shape 的可能性。
