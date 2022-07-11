### keyof类型操作符

对一个对象类型使用 keyof 操作符，会返回该对象属性名组成的一个字符串或者数字字面量的联合。

```ts

type Point = { x: number; y: number };
type P = keyof Point;

// type P = keyof Point
// 类型 P 就等同于 "x" | "y"

```

但如果这个类型有一个 string 或者 number 类型的索引签名，keyof 则会直接返回这些类型：

```ts

type Arrayish = { [n: number]: unknown };
type A = keyof Arrayish;
// type A = number

type Mapish = { [k: string]: boolean };
type M = keyof Mapish;
// type M = string | number

```

注意在这个例子中，M 是 string | number，这是因为 JavaScript 对象的属性名会被强制转为一个字符串，所以 obj[0] 和 obj["0"] 是一样的。

