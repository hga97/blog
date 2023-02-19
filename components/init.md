组件库，主要为了提高开发效率，避免开发同学复制粘贴和重复造轮子。

组件库 to do list：

- 项目初始化：开发前期准备。eslint/commit lint/typescript
- 开发阶段：使用 dumi 进行开发调试以及文档编写
- 打包阶段：输出 umd/cjs/esm 产物并支持按需加载
- 组件测试： 使用@testing-library/react 及其相关生态进行组件测试
- 发布 npm：编写脚本完成发布
- 部署文档站点：使用 github pages 以及 github actions 完成文档站点自动部署

### 代码规范

#### 使用 @umijs/fabric 的配置

```zsh
yarn add @umijs/fabric --dev

yarn add prettier --dev # 因为@umijs/fabric没有将prettier作为依赖 所以我们需要手动安装
```

.eslintrc.js

```js
module.exports = {
  extends: [require.resolve("@umijs/fabric/dist/eslint")],
};
```

.prettierrc.js

```js
const fabric = require("@umijs/fabric");

module.exports = {
  ...fabric.prettier,
};
```

.stylelintrc.js

```js
module.exports = {
  extends: [require.resolve("@umijs/fabric/dist/stylelint")],
};
```

#### eslint-config-prettier

安装 eslint-config-prettier，把 Prettier 推荐的格式问题的配置以 ESLint rules 的方式写入。

```zsh
yarn add eslint-plugin-prettier --dev
```

.eslintrc.js

```js
module.exports = {
  plugins: ["prettier"],
  rules: {
    "prettier/prettier": "error",
  },
};

// 相当于
{
  "extends": ["plugin:prettier/recommended"]
}
```

### Commit Lint

#### pre-commit 代码规范检测

lint-staged，能够让 lint 只检测暂存区的文件

```zsh
yarn add lint-staged --dev
```

package.json

```json
"lint-staged": {
  "src/**/*.ts?(x)": [
    "prettier --write",
    "eslint --fix",
    "git add"
  ],
  "src/**/*.less": [
    "stylelint --syntax less --fix",
    "git add"
  ]
},
```

husky, 方便在项目中添加 git hooks

```zsh
yarn add husky --dev
```

```zsh
npm pkg set scripts.prepare="husky install"
npm run prepare
```

```zsh
npx husky add .husky/pre-commit "npm test"
git add .husky/pre-commit
```

调用 git commit 命令时执行 lint-staged 进行代码检测

.husky/pre-commit

```
npx lint-staged
```

#### Commit Message 检测

commitlint，提交信息规范

```zsh
# Install commitlint cli and conventional config
npm install --save-dev @commitlint/{config-conventional,cli}
# For Windows:
npm install --save-dev @commitlint/config-conventional @commitlint/cli

# Configure commitlint to use conventional config
echo "module.exports = {extends: ['@commitlint/config-conventional']}" > commitlint.config.js
```

git commit 提交的信息使用 commitlint 校验

```zsh
npx husky add .husky/commit-msg  'npx --no -- commitlint --edit ${1}'
```

#### 标准化的 commit message

Commitizen 是一个支持 angular 的 Commit message 格式，其提供了一个 git cz 的命令来替代 git commit。

```zsh
yarn add commitizen --dev
```

changelog

```zsh
yarn add cz-conventional-changelog --dev
```

### TypeScript

```zsh
yarn add typescript --dev
```

tsconfig.json

```json
{
  "compilerOptions": {
    "baseUrl": "./",
    "target": "esnext",
    "module": "commonjs",
    "jsx": "react",
    "declaration": true,
    "declarationDir": "lib",
    "strict": true,
    "moduleResolution": "node",
    "allowSyntheticDefaultImports": true,
    "esModuleInterop": true,
    "resolveJsonModule": true
  },
  "include": ["src", "typings.d.ts"],
  "exclude": ["node_modules"]
}
```
