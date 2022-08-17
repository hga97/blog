### 是什么？

webpack-bundle-analyzer 是 webpack 的插件，需要配合 webpack 和 webpack-cli 一起使用。这个插件可以读取输出文件夹（通常是 dist）中的 stats.json 文件，把该文件可视化展现，生成代码分析报告，可以直观地分析打包出的文件有哪些，及它们的大小、占比情况、各文件 Gzipped 后的大小、模块包含关系、依赖项等，对应做出优化，从而帮助提升代码质量和网站性能。

### 安装并配置

安装 webpack-bundle-analyzer

```zsh
# NPM
npm install --save-dev webpack-bundle-analyzer
# Yarn
yarn add -D webpack-bundle-analyzer
```

配置 webpack.config.js 文件

```js
const { BundleAnalyzerPlugin } = require("webpack-bundle-analyzer");

module.exports = {
  plugins: [
    // ... 其他 plugin 配置
    process.env.ANALYZER && new BundleAnalyzerPlugin(), // 使用默认配置
  ],
};
```

配置 package.json 文件

```json
"scripts": {
    "analyzer": "cross-env NODE_ENV=prod webpack --env mode=production analyzer",
},
```

### 包体积优化实践

#### 运行指令，看相关模块的依赖的组成

```zsh
 yarn analyzer
```

浏览器会自动打开 127.0.0.1:8888，模块的依赖如下

![在这里插入图片描述](https://img-blog.csdnimg.cn/cafc75af5aae4068b7c60818a687844d.png)

#### 分析问题

##### lodash 和 moment 包，占比大

![在这里插入图片描述](https://img-blog.csdnimg.cn/8df5e67bf1f64b4d8d009e8ba6cdf831.png)

只使用了 padStart 一个函数，打包出来就很大。

然后发现原因是：没有使用按需引入，默认会把整个包都打包进项目里的

解决：

```tsx
import padStart from "lodash/padStart"; // 按需加载
```

moment 的包比较大，可以使用 day.js 替代，和 Moment.js 有着相同的 API 和模式，超小的压缩体积，仅仅有 2kb 左右。

解决：

```tsx
import dayjs from "dayjs"; // day.js
```

解决前后对比

![在这里插入图片描述](https://img-blog.csdnimg.cn/1bdd674408ba46cc83d3da5824de8171.png)

![在这里插入图片描述](https://img-blog.csdnimg.cn/45374127aed54c08be1332ecec491386.png)

##### 依赖项不使用，但项目引用到

```tsx
import { Text } from "../Text/index"; // Text组件使用到intl，但组件在项目中不再使用

// Text.tsx
import intl from "react-intl-universal";
```

解决： 删除掉

![在这里插入图片描述](https://img-blog.csdnimg.cn/0a12af5030a44f4ead06e517c8bcd339.png)




