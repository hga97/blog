### typeof 类型操作符

JavaScript 本身就有 typeof 操作符，可以在表达式上下文中使用

```js
console.log(typeof "Hello world"); //string
```

而 TypeScript 添加 typeof 方法可以在类型上下文中使用，用于获取一个变量或者属性的类型

```ts
let s = "hello";
let n: typeof s; // let s: string

const defaultConfig = {
  throwOnError: false,
};

// const a: {
//     throwOnError: boolean;
// }
const a: typeof defaultConfig;
```
