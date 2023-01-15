#### require.resolve

查询某个模块的完整路径

<!-- https://juejin.cn/post/6844904055806885895 -->

1、模块路径

```ts
require.resolve("@typescript-eslint/parser");

/Users/xxxx/Desktop/FE/project/h-cli/node_modules/@typescript-eslint/parser/dist/index.js

// 当前文件夹node_modules模块里找，找到@typescript-eslint/parser，
   parser的package.json文件main属性为dist/index.js，找到对应的文件。
```

#### path.resolve

1、\_\_dirname

当前文件所在文件夹的绝对路径

```ts
console.log(__dirname);// /Users/xxxxx/Desktop/FE/project/h-cli/lib/util

resolve(__dirname, '../../node_modules') // /Users/xxxxx/Desktop/FE/project/h-cli/node_modules
```

2、process.cwd()

当前执行 node 命令时候的文件夹地址

```ts
console.log(process.cwd()); // /Users/xxxxx/Desktop/FE/project/react-tpl

resolve(process.cwd(), "src"); // /Users/xxxxx/Desktop/FE/project/react-tpl/src

```
