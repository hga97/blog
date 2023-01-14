### 介绍

ESLint 是一个用来识别 ECMAScript/JavaScript 并且按照规则给出报告的代码检测工具，使用它可以避免低级错误和统一代码的风格。

在许多方面，它和 JSLint、JSHint 相似，除了少数的例外：

ESLint 使用 Espree 解析 JavaScript。  
ESLint 使用 AST 去分析代码中的模式  
ESLint 是完全插件化的。每一个规则都是一个插件并且你可以在运行时添加更多的规则。

### 使用

#### 安装

```zsh
npm i -g eslint //全局安装
npm i -d eslint //局部安装
```

#### 初始化

```zsh
eslint --init
```

根据项目需要选择自己的规范，选择完成后自动安装其需要的插件。并生成.eslintrc.js 文件。

#### 开发环境校验

vscode 安装 eslint 插件，在我们编写代码时，eslint 插件会自动纠正我们错误的写法。

### 制作统一的 eslint 校验模块

方案：将包作为命令行工具，安装 npm 包，在项目输入指令，格式化该项目。

#### Node ESLint API

```tsx
const eslint = new ESLint(options); // 创建一个eslint实例
```

options 对象配置参数解释：

**file enumeration**

- extensions

  扩展名，eslint 只检查给定的目录中包含扩展名的文件。

**Linting**

- useEslintrc

  为 true，eslint 不会加载配置文件 .eslintrc 文件，只有构造函数的 option 配置有效。

- overrideConfig

  默认为空，项目的 eslint 配置覆盖此实例使用的所有配置。  
  你可以使用此选项定义将要使用的设置，即使项目的 eslint 对其进行了配置。（覆盖本地的配置）

  - env

    启用的环境，并设置它们为 true。

  - extends

    extends 可以看做是去集成一个个配置方案的最佳实践。

  - parser

    解析器

  - parserOptions

    启用对 ECMAScript 其它版本和 JSX 的支持。

    ecmaFeatures：想使用的额外的语言特性，jsx；  
    ecmaVersion：ECMAScript 版本；
    sourceType：有两个值，script 和 module。对于 ES6+ 的语法和用 import / export 的语法必须用 module.

  - plugins

    plugin 插件主要是为 eslint 新增一些检查规则。

- resolvePluginsRelativeTo

  如果存在路径，ESLint 将从那里加载所有插件。

**Autofix**

- fix

  为 true， eslint.lintFiles() 与 eslint.lintText()工作在 autofix（自动修复）模式。

#### 创建 eslint 工具类

<!-- https://eslint.org/docs/latest/developer-guide/nodejs-api -->

```ts
import { ESLint } from "eslint";

const eslint = new ESLint({
  fix: true,
  extensions: [".js", ".ts"], // 扩展
  useEslintrc: false,
  overrideConfig: {
    env: {
      browser: true,
      es2021: true,
    },
    extends: ["eslint:recommended"],
    parser: require.resolve("@typescript-eslint/parser"),
    parserOptions: {
      ecmaFeatures: {
        jsx: true,
      },
      ecmaVersion: 12,
      sourceType: "module",
    },
    plugins: ["react", "@typescript-eslint"],
  }, // 覆盖配置
  resolvePluginsRelativeTo: getDirPath("node_modules"), //指定 loader 加载路径
});
```
