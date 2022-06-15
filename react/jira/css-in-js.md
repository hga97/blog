### antd 进行主题配置

?>
CRA 此前设计的思路理解：抽象出 react 应用构建过程中最核心的部分，这部分通常为 Webpack 配置；业界此前一般是抽象为工程模板(Boilerplate)，一般由 git fork / cli 文件复制完成；但 CRA 进一步将其封装为 npm 包，使得其升级过程变得可控。  
但某种程度上，他默认提供的方案违背了开闭原则: 如果我想要对其扩展，必须修改其源码（比如 fork 一份自己的 react-scripts），或者执行 eject 再修改。无论哪种方式都会导致后续升级变得困难。

需要对 create-react-app 的默认配置进行自定义，使用 craco （一个对 create-react-app 进行自定义配置的社区解决方案）。
[craco](https://zhuanlan.zhihu.com/p/423919309)

```js

yarn add @craco/craco

/* package.json */
"scripts": {
-   "start": "react-scripts start",
-   "build": "react-scripts build",
-   "test": "react-scripts test",
+   "start": "craco start",
+   "build": "craco build",
+   "test": "craco test",
}

```

然后在项目根目录创建一个 craco.config.js 用于修改默认配置。

```js
const CracoLessPlugin = require("craco-less");

module.exports = {
  plugins: [
    {
      plugin: CracoLessPlugin,
      options: {
        lessLoaderOptions: {
          lessOptions: {
            modifyVars: { "@primary-color": "#1DA57A" },
            javascriptEnabled: true,
          },
        },
      },
    },
  ],
};
```

### css-in-js

#### 传统 css 的缺陷

1、缺乏模块组织

传统的 js 和 css 都没有模块的概念，后来 js 有了 commonJs 和 ECMAScript Module
css-in-js 可以使用模块化的方式组织 css，依托 js 的模块方案

```ts
improt styled from '@emotion/styled'

export const Button = styled.button`
  color: red
`
```

2、缺乏作用域

传统的 css 只有一个全局作用域，一个 class 可以匹配全局的任意元素。
随着项目成长，css 会变得越来越难以组织。

css-in-js 可以通过生成独特的选择符，来实现作用域的效果

3、隐式依赖，让样式难以追踪

```css
.target .name h1 {
  color: red;
}

body #container h1 {
  color: green;
}
```

```html
<!DOCTYPE html>
<html lang="en">
  <body>
    <div id="container">
      <div class="target">
        <div class="name">
          <h1>我是啥颜色？(绿色)</h1>
        </div>
      </div>
    </div>
  </body>
</html>
```

而css-in-js的方案就是简单直接、易于追踪

```ts
export const Title = style.h1`
  color: green;
`

<Title>我是啥颜色？</Title>
```

4、没有变量
传统的css规则里没有变量，但是css-in-js中可以方便地控制变量

```ts
const Container = styled.div(props => ({
  display: 'flex',
  flexDirection: props.column && 'column'
}))
```

5、css选择器与html元素耦合

改动一个标签，需要同时改动css和html。
而css-in-js中，html和css是结合在一起的，易于修改。


#### Emotion介绍