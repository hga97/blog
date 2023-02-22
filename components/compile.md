宿主环境各不相同，需要将源码进行相关处理后发布至 npm。

to do list

- 导出类型声明文件
- 导出 cjs / esm 形式的产物供使用者引入
- 支持样式文件 css 引入，而非只有 less，减少业务方接入成本
- 支持按需加载

### 导出类型声明文件

能得到自动提示，也能够复用相关组件的类型定义。

package.json

```json
{
  "typings": "lib/index.d.ts", // 定义类型入口文件
  "scripts": {
    "build:types": "tsc -p tsconfig.build.json && cpr lib esm" // 执行tsc命令生成类型声明文件
  }
}
```

tsconfig.build.json

```json
{
  "extends": "./tsconfig.json", // 扩展tsconfig.json文件
  "compilerOptions": { "emitDeclarationOnly": true }, // 只生成声明文件
  "exclude": ["**/__tests__/**", "**/demo/**", "node_modules", "lib", "esm"] // 排除示例、测试以及打包好的文件夹
}
```

cpr 将 lib 的声明文件拷贝了一份，并将文件夹重命名为 esm，
用于后面存放 esm 形式的组件。保证用户手动按需引入组件时依旧可以获取自动提示。

### 导出 cjs 模块

导出 cjs 版本原因，es 模块兼容性比较差，如服务端渲染。

```zsh
yarn add @babel/core @babel/preset-env @babel/preset-react @babel/preset-typescript
@babel/plugin-proposal-class-properties  @babel/plugin-transform-runtime --dev
```

1、.browserslistrc

为流行的 JavaScript 工具（如 Autoprefixer、Babel、ESLint、PostCSS 和 Webpack）共享浏览器兼容性配置

- babel，在 @babel/preset-env 中使用 core-js 作为垫片
- postcss 使用 autoprefixer 作为垫片

```js
module.exports = {
  // 智能预设：能够编译ES6语法
  presets: [
    [
      "@babel/preset-env",
      // 按需加载core-js的polyfill
      { useBuiltIns: "usage", corejs: { version: "3", proposals: true } },
    ],
  ],
};
```

前端打包体积与垫片关系

1、由于低浏览器版本的存在，垫片是必不可少的
2、垫片越少，则打包体积越小
3、浏览器版本越新，则垫片越少

2、polyfill

建议将 polyfill 的选择权交还给使用者，在宿主环境进行 polyfill

3、.babelrc.js

babel 是一种 js 语法编译器，在前端开发过程中，由于浏览器的版本和兼容性问题，很多 js 的新方法和特性的使用都受到了限制。
使用 babel 可以将代码中 js 代码编译成兼容绝大多数主流浏览器的代码。

.babelrc 文件需要的配置项主要有预设(presets)和插件(plugins)。

```js
module.exports = {
  presets: ["@babel/env", "@babel/typescript", "@babel/react"],
  plugins: [
    "@babel/plugin-transform-runtime",
    "@babel/proposal-class-properties",
  ],
};
```

4、gulpfile.js

```js
const gulp = require("gulp");
const babel = require("gulp-babel");

const paths = {
  dest: {
    lib: "lib", // commonjs 文件存放的目录名 - 本块关注
    esm: "esm", // ES module 文件存放的目录名 - 暂时不关心
    dist: "dist", // umd文件存放的目录名 - 暂时不关心
  },
  styles: "src/**/*.less", // 样式文件路径 - 暂时不关心
  scripts: ["src/**/*.{ts,tsx}", "!src/**/demo/*.{ts,tsx}"], // 脚本文件路径
};

function compileCJS() {
  const { dest, scripts } = paths;
  return gulp
    .src(scripts)
    .pipe(babel()) // 使用gulp-babel处理
    .pipe(gulp.dest(dest.lib));
}

const build = gulp.parallel(compileCJS);

exports.build = build;

exports.default = build;
```

### 导出 esm 模块

如果使用 import 对该库进行导入，则首次寻找 module 字段引入，否则引入 main 字段。

module 字段作为 es module 入口  
main 字段作为 commonjs 入口

package.json

```json
  "main": "lib/index.js",
  "module": "esm/index.js",
```

1、.babelrc.js

```js
module.exports = {
  presets: [
    [
      "@babel/env",
      {
        modules: false, // 关闭模块转换
      },
    ],
    "@babel/typescript",
    "@babel/react",
  ],
  plugins: [
    "@babel/proposal-class-properties",
    [
      "@babel/plugin-transform-runtime",
      {
        useESModules: true, // 使用esm形式的helper
      },
    ],
  ],
};
```

使用环境变量区分 esm 和 cjs（执行任务时设置对应的环境变量即可)

```js
module.exports = {
  presets: ["@babel/env", "@babel/typescript", "@babel/react"],
  plugins: [
    "@babel/plugin-transform-runtime",
    "@babel/proposal-class-properties",
  ],
  env: {
    esm: {
      presets: [
        [
          "@babel/env",
          {
            modules: false,
          },
        ],
      ],
      plugins: [
        [
          "@babel/plugin-transform-runtime",
          {
            useESModules: true,
          },
        ],
      ],
    },
  },
};
```

2、gulpfile.js

```js
function compileScripts(babelEnv, destDir) {
  const { scripts } = paths;
  process.env.BABEL_ENV = babelEnv;

  return gulp.src(scripts).pipe(babel()).pipe(gulp.dest(destDir));
}

function compileCJS() {
  const { dest } = paths;
  return compileScripts("cjs", dest.lib);
}

function compileESM() {
  const { dest } = paths;
  return compileScripts("esm", dest.esm);
}

const buildScripts = gulp.series(compileCJS, compileESM);

const build = gulp.parallel(buildScripts);
```

### 处理样式

1、拷贝 less 文件

会将 less 文件包含在 npm 包中，用户可以通过 /lib/alert/style/index.js 的形式按需引入 less 文件，此处可以直接将 less 文件拷贝至目标文件夹。

gulpfile.js

```js
function copyLess() {
  return gulp
    .src(paths.styles)
    .pipe(gulp.dest(paths.dest.lib))
    .pipe(gulp.dest(paths.dest.esm));
}

const build = gulp.parallel(buildScripts, copyLess);
```

2、预处理器问题

若使用者没有使用 less 预处理器，使用的是 sass 方案甚至原生 css 方案，那现有方案就搞不定了。

方案

- 告知业务方增加 less-loader。会导致业务方使用成本增加；
- 打包出一份完整的 css 文件，进行全量引入。无法进行按需引入；
- css in js 方案；
- 提供一份 style/css.js 文件，引入组件 css 样式依赖，而非 less 依赖，组件库底层抹平差异。（antd 方案）

3、生成 css 文件

```zsh
yarn add gulp-less gulp-autoprefixer gulp-cssnano --dev
```

gulpfile.js

```js
function less2css() {
  return gulp
    .src(paths.styles)
    .pipe(less()) // 处理less文件
    .pipe(autoprefixer()) // 根据browserslistrc增加前缀
    .pipe(cssnano({ zindex: false, reduceIdents: false })) // 压缩
    .pipe(gulp.dest(paths.dest.lib))
    .pipe(gulp.dest(paths.dest.esm));
}

const build = gulp.parallel(buildScripts, copyLess, less2css);
```

4、生成 css.js

需要一个 alert/style/css.js 来帮用户引入 css 文件。

在处理 scripts 任务中，截住 style/index.js，生成 style/css.js，并通过正则将引入的 less 文件后缀改成 css。

gulpfile.js

```js
function cssInjection(content) {
  return content
    .replace(/\/style\/?'/g, "/style/css'")
    .replace(/\/style\/?"/g, '/style/css"')
    .replace(/\.less/g, ".css");
}

function compileScripts(babelEnv, destDir) {
  const { scripts } = paths;
  process.env.BABEL_ENV = babelEnv;
  return gulp
    .src(scripts)
    .pipe(babel()) // 使用gulp-babel处理
    .pipe(
      through2.obj(function z(file, encoding, next) {
        this.push(file.clone());
        // 找到目标
        if (file.path.match(/(\/|\\)style(\/|\\)index\.js/)) {
          const content = file.contents.toString(encoding);
          file.contents = Buffer.from(cssInjection(content)); // 文件内容处理
          file.path = file.path.replace(/index\.js/, "css.js"); // 文件重命名
          this.push(file); // 新增该文件
          next();
        } else {
          next();
        }
      })
    )
    .pipe(gulp.dest(destDir));
}
```

### 按需加载

在 package.json 中增加 sideEffects 属性，配合 ES module 达到 tree shaking 效果（将样式依赖文件标注为 side effects，避免被误删除）。

1、手动按需引入

```tsx
import { Alert } from "h-ui";
import "h-ui/esm/alert/style";
```

2、babel-plugin-import（增加了使用成本）

```tsx
import { Alert } from "h-ui";
```

.babelrc.js

```js
{
  "plugins": ['import', {
    "libraryName": "h-ui",
    "libraryDirectory": "es",
    "style": "css"
  }]
}
```
