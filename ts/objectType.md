### 对象类型

在 js 中，最基本的将数据成组和分发的方式就是通过对象。在 ts 中，通过对象类型来描述对象。

对象类型可以是匿名的

```ts
function greet(person: { name: string; age: number }) {
  return "Hello " + person.name;
}
```

也可以使用接口进行定义：

```ts
interface Person {
  name: string;
  age: number;
}

function greet(person: Person) {
  return "Hello " + person.name;
}
```

或者通过类型别名：

```ts
type Person = {
  name: string;
  age: number;
};

function greet(person: Person) {
  return "Hello " + person.name;
}
```

### 属性修饰符

对象类型中的每个属性可以说明它的类型、属性是否可选、属性是否只读等信息。

#### 可选属性

可以在属性名后面加一个 ? 标记表示这个属性是可选的：

```ts
interface PaintOptions {
  shape: Shape;
  xPos?: number;
  yPos?: number;
}

function paintShape(opts: PaintOptions) {
  // ...
}

const shape = getShape();
paintShape({ shape });
paintShape({ shape, xPos: 100 });
paintShape({ shape, yPos: 100 });
paintShape({ shape, xPos: 100, yPos: 100 });
```

也可以尝试读取这些属性

```ts
function paintShape(opts: PaintOptions) {
  // const xPos: number | undefined;
  let xPos = opts.xPos;
  // const yPos: number | undefined;
  let yPos = opts.yPos;
}
```

如果一个属性值没有被设置，我们获取会得到 undefined

```ts
function paintShape(opts: PaintOptions) {
  let xPos = opts.xPos === undefined ? 0 : opts.xPos;
  // let xPos: number
  let yPos = opts.yPos === undefined ? 0 : opts.yPos;
  // let yPos: number
}
```

这种判断在 JavaScript 中很常见，以至于提供了专门的语法糖：

```ts
function paintShape({ shape, xPos = 0, yPos = 0 }: PaintOptions) {
  console.log("x coordinate at", xPos); // (parameter) xPos: number
  console.log("y coordinate at", yPos); // (parameter) yPos: number
  // ...
}
```

使用了解构语法以及为 xPos 和 yPos 提供了默认值。现在 xPos 和 yPos 的值在 paintShape 函数内部一定存在，但对于 paintShape 的调用者来说，却是可选的。

?> 注意现在并没有在解构语法里放置类型注解的方式。这是因为在 JavaScript 中，下面的语法代表的意思完全不同。

```ts
function draw({ shape: Shape, xPos: number = 100 }) {
  // 找不到名称“shape”。你是否指的是“Shape”?
  render(shape);

  render(xPos);
}
```

在对象解构语法中，shape: Shape 表示的是把 shape 的值赋值给局部变量 Shape。 xPos: number 也是一样，会基于 xPos 创建一个名为 number 的变量。

#### readonly 属性

在 TypeScript 中，属性可以被标记为 readonly，这不会改变任何运行时的行为，但在类型检查的时候，一个标记为 readonly 的属性是不能被写入的。

```ts
interface SomeType {
  readonly prop: string;
}

function doSomething(obj: SomeType) {
  console.log(`prop has the value '${obj.prop}'.`);
  obj.prop = "hello"; // 无法分配到 "prop" ，因为它是只读属性。
}
```

不过使用 readonly 并不意味着一个值就完全是不变的，亦或者说，内部的内容是不能变的。readonly 仅仅表明属性本身是不能被重新写入的。

```ts
interface Home {
  readonly resident: { name: string; age: number };
}

function visitForBirthday(home: Home) {
  // We can read and update properties from 'home.resident'.
  console.log(`Happy birthday ${home.resident.name}!`);
  home.resident.age++;
}

function evict(home: Home) {
  // But we can't write to the 'resident' property itself on a 'Home'.
  home.resident = {
    // 无法分配到 "resident" ，因为它是只读属性。
    name: "Victor the Evictor",
    age: 42,
  };
}
```

TypeScript 在检查两个类型是否兼容的时候，并不会考虑两个类型里的属性是否是 readonly，这就意味着，readonly 的值是可以通过别名修改的。

```ts
interface Person {
  name: string;
  age: number;
}

interface ReadonlyPerson {
  readonly name: string;
  readonly age: number;
}

let writablePerson: Person = {
  name: "Person McPersonface",
  age: 42,
};

// works
let readonlyPerson: ReadonlyPerson = writablePerson;

console.log(readonlyPerson.age); // prints '42'
writablePerson.age++;
console.log(readonlyPerson.age); // prints '43'
```

#### 索引签名

略

### 属性继承

BasicAddress 类型用来描述在美国邮寄信件和包裹的所需字段。

```ts
interface BasicAddress {
  name?: string;
  street: string;
  city: string;
  country: string;
  postalCode: string;
}
```

同一个地址的建筑往往还有不同的单元号，我们可以再写一个 AddressWithUnit

```ts
interface AddressWithUnit {
  name?: string;
  unit: string;
  street: string;
  city: string;
  country: string;
  postalCode: string;
}
```

这样写固然可以，但为了加一个字段，就是要完全的拷贝一遍。

我们可以改成继承 BasicAddress 的方式来实现：

```ts
interface BasicAddress {
  name?: string;
  street: string;
  city: string;
  country: string;
  postalCode: string;
}

interface AddressWithUnit extends BasicAddress {
  unit: string;
}
```

对接口使用 extends 关键字允许我们有效的从其他声明过的类型中拷贝成员，并且随意添加新成员。

接口也可以继承多个类型：

```ts
interface Colorful {
  color: string;
}

interface Circle {
  radius: number;
}

interface ColorfulCircle extends Colorful, Circle {}

const cc: ColorfulCircle = {
  color: "red",
  radius: 42,
};
```

### 交叉类型

TypeScript 也提供了名为交叉类型的方法，用于合并已经存在的对象类型。

交叉类型的定义需要用到 & 操作符：

```ts
interface Colorful {
  color: string;
}
interface Circle {
  radius: number;
}

type ColorfulCircle = Colorful & Circle;
```

这里，我们连结 Colorful 和 Circle 产生了一个新的类型，新类型拥有 Colorful 和 Circle 的所有成员。

```ts
function draw(circle: Colorful & Circle) {
  console.log(`Color was ${circle.color}`);
  console.log(`Radius was ${circle.radius}`);
}

// okay
draw({ color: "blue", radius: 42 });

// 类型“{ color: string; raidus: number; }”的参数不能赋给类型“Colorful & Circle”的参数。
// oops
draw({ color: "red", raidus: 42 });
```

### 接口继承与交叉类型

使用继承的方式，如果重写类型会导致编译错误，但交叉类型不会：

```ts
interface Colorful {
  color: string;
}

// 接口“ColorfulSub”错误扩展接口“Colorful”。
// 属性“color”的类型不兼容。不能将类型“number”分配给类型“string”。

interface ColorfulSub extends Colorful {
  color: number;
}
```

```ts
// 可以重写

interface Colorful {
  color: string;
}

type ColorfulSub = Colorful & {
  color: number;
};

// color: never
// 不能将类型“string”分配给类型“never”。ts(2322)
const a: ColorfulSub = {
  color: "asd",
};
```

虽然不会报错，那 color 属性的类型是什么呢，答案是 never，取得是 string 和 number 的交集。

### 泛型对象类型

创建一个泛型 Box ，它声明了一个类型参数

```ts
interface Box<Type> {
  contents: Type;
}
```

Box 的 Type 就是 contents 拥有的类型 Type

当引用 Box 的时候，需要给予一个类型实参替换掉 Type

```ts
let box: Box<string>;
```

Box\<Type\> 的 Type 为 string, 最后的结果就会变成 { contents: string }

需要一个新的类型，完全不需要再重新声明一个类型

```ts
interface Box<Type> {
  contents: Type;
}

interface Apple {
  // ....
}

// Same as '{ contents: Apple }'.
type AppleBox = Box<Apple>;
```

意味着可以利用泛型函数避免使用函数重载。

```ts
function setContents(box: StringBox, newContents: string): void;
function setContents(box: NumberBox, newContents: number): void;
function setContents(box: BooleanBox, newContents: boolean): void;
function setContents(box: { contents: any }, newContents: any) {
  box.contents = newContents;
}

function setContents<Type>(box: Box<Type>, newContents: Type) {
  box.contents = newContents;
}
```

类型别名也是可以使用泛型的。比如：

```ts
type Box<Type> = {
  contents: Type;
};
```

类型别名不同于接口，可以描述的不止是对象类型，所以也可以用类型别名写一些其他种类的泛型帮助类型。

```ts
ttype OrNull<Type> = Type | null;

type OneOrMany<Type> = Type | Type[];

type OneOrManyOrNull<Type> = OrNull<OneOrMany<Type>>;

type OneOrManyOrNull<Type> = OneOrMany<Type> | null

type OneOrManyOrNullStrings = OneOrManyOrNull<string>;

type OneOrManyOrNullStrings = OneOrMany<string> | null

```

#### Array 类型

类型 number[] 或者 string[] 的时候，其实它们只是 Array\<number\> 和 Array\<string\> 的简写形式而已。

```ts
function doSomething(value: Array<string>) {
  // ...
}

let myArray: string[] = ["hello", "world"];

// either of these work!
doSomething(myArray);
doSomething(new Array("hello", "world"));
```

类似于上面的 Box 类型，Array 本身就是一个泛型：

```ts
interface Array<Type> {
  /**
   * Gets or sets the length of the array.
   */
  length: number;

  /**
   * Removes the last element from an array and returns it.
   */
  pop(): Type | undefined;

  /**
   * Appends new elements to an array, and returns the new length of the array.
   */
  push(...items: Type[]): number;

  // ...
}
```

现代 JavaScript 也提供其他是泛型的数据结构，比如 Map\<K, V\> ， Set\<T\> 和 Promise\<T\>。因为 Map 、Set 、Promise 的行为表现，它们可以跟任何类型搭配使用。

#### ReadonlyArray 类型

ReadonlyArray 是一个特殊类型，它可以描述数组不能被改变。

```ts
function doStuff(values: ReadonlyArray<string>) {
  const copy = values.slice();
  console.log(`The first value is ${values[0]}`);

  // 类型“readonly string[]”上不存在属性“push”。ts(2339)
  values.push("hello!");
}
```

ReadonlyArray 主要是用来做意图声明。当我们看到一个函数返回 ReadonlyArray，就是在告诉我们不能去更改其中的内容，当我们看到一个函数支持传入 ReadonlyArray ，这是在告诉我们可以放心的传入数组到函数中，而不用担心会改变数组的内容。

不像 Array，ReadonlyArray 并不是一个我们可以用的构造器函数。

```ts
new ReadonlyArray("red", "green", "blue");
// “ReadonlyArray”仅表示类型，但在此处却作为值使用。
```

可以直接把一个常规数组赋值给 ReadonlyArray

```ts
const roArray: ReadonlyArray<string> = ["red", "green", "blue"];
```

TypeScript 也针对 ReadonlyArray\<Type\> 提供了更简短的写法 readonly Type[]。

```ts
function doStuff(values: readonly string[]) {
  const copy = values.slice();
  console.log(`The first value is ${values[0]}`);

  // 类型“readonly string[]”上不存在属性“push”。ts(2339)
  values.push("hello!");
}
```

Arrays 和 ReadonlyArray 并不能双向的赋值：

```ts
let x: readonly string[] = [];
let y: string[] = [];

x = y; // ok
y = x; // 类型 "readonly string[]" 为 "readonly"，不能分配给可变类型 "string[]"。
```

#### 元组类型

元组类型是另外一种 Array 类型，当你明确知道数组包含多少个元素，并且每个位置元素的类型都明确知道的时候，就适合使用元组类型。

```ts
type StringNumberPair = [string, number];
```

StringNumberPair 就是 string 和 number 的元组类型。

StringNumberPair 描述了一个数组，索引 0 的值的类型是 string，索引 1 的值的类型是 number。

```ts
function doSomething(pair: [string, number]) {
  const a = pair[0];

  const a: string;
  const b = pair[1];

  const b: number;
  // ...
}

doSomething(["hello", 42]);
```

如果要获取元素数量之外的元素，TypeScript 会提示错误：

```ts
function doSomething(pair: [string, number]) {
  const c = pair[2];
  // 长度为 "2" 的元组类型 "[string, number]" 在索引 "2" 处没有元素。
}
```

我们也可以使用 JavaScript 的数组解构语法解构元组：

```ts
function doSomething(stringHash: [string, number]) {
  const [inputString, hash] = stringHash;
  console.log(inputString); // const inputString: string
  console.log(hash); // const hash: number
}
```

除了长度检查，简单的元组类型跟声明了 length 属性和具体的索引属性的 Array 是一样的。

```ts
interface StringNumberPair {
  // specialized properties
  length: 2;
  0: string;
  1: number;

  // Other 'Array<string | number>' members...
  slice(start?: number, end?: number): Array<string | number>;
}
```

在元组类型中，你也可以写一个可选属性，但可选元素必须在最后面，而且也会影响类型的 length 。

```ts
type Either2dOr3d = [number, number, number?];

function setCoordinate(coord: Either2dOr3d) {
  const [x, y, z] = coord;
  // const z: number | undefined

  console.log(`Provided coordinates had ${coord.length} dimensions`);
  // (property) length: 2 | 3
}
```

Tuples 也可以使用剩余元素语法，但必须是 array/tuple 类型：

```ts
type StringNumberBooleans = [string, number, ...boolean[]];
type StringBooleansNumber = [string, ...boolean[], number];
type BooleansStringNumber = [...boolean[], string, number];
```

有剩余元素的元组并不会设置 length，因为它只知道在不同位置上的已知元素信息：

```ts
const a: StringNumberBooleans = ["hello", 1];
const b: StringNumberBooleans = ["beautiful", 2, true];
const c: StringNumberBooleans = ["world", 3, true, false, true, false, true];

console.log(a.length); // (property) length: number

type StringNumberPair = [string, number];
const d: StringNumberPair = ["1", 1];
console.log(d.length); // (property) length: 2
```

可选元素和剩余元素的存在，使得 TypeScript 可以在参数列表里使用元组，就像这样：

```ts
function readButtonInput(...args: [string, number, ...boolean[]]) {
  const [name, version, ...input] = args;
  // ...
}
```

基本等同于：

```ts
function readButtonInput(name: string, version: number, ...input: boolean[]) {
  // ...
}
```

#### readonly 元组类型

元组类型也是可以设置 readonly 的，这样 TypeScript 就不会允许写入 readonly 元组的任何属性：

```ts
function doSomething(pair: readonly [string, number]) {
  pair[0] = "hello!";
  // Cannot assign to '0' because it is a read-only property.
}
```

在大部分的代码中，元组只是被创建，使用完后也不会被修改，所以尽可能的将元组设置为 readonly 是一个好习惯。

如果我们给一个数组字面量 const 断言，也会被推断为 readonly 元组类型。

```ts
let point = [3, 4] as const;

function distanceFromOrigin([x, y]: [number, number]) {
  return Math.sqrt(x ** 2 + y ** 2);
}

// 类型“readonly [3, 4]”的参数不能赋给类型“[number, number]”的参数。
// 类型 "readonly [3, 4]" 为 "readonly"，不能分配给可变类型 "[number, number]"。
distanceFromOrigin(point);
```

因为 point 的类型被推断为 readonly [3, 4]，它跟 [number number] 并不兼容
