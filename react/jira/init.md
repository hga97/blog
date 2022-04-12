### cli
Create React App是一种官方支持的创建单页React应用程序的方法。它提供了一个没有配置的现代构建设置。

```bash
npx create-react-app my-app --template typescript
使用ts模版
```

?> npx  
避免全局安装，执行一次性命令

### 规范工程

#### eslint
create-react-app自带了eslint配置，主要解决的是代码质量问题。  

#### prettier 
主要解决的是代码风格问题。  

eslint-config-prettier
使用eslint-config-prettier来关掉(disable)所有和Prettier冲突的ESLint的配置，
把Prettier推荐的格式问题的配置以ESLint rules的方式写入。
```bash
eslint配置  
{
  "extends": [
    "prettier"
  ]
}
```


#### husky、lint-staged和commitlint
husky：可以方便在项目中添加git hooks

```bash
安装
npm install -D husky  

package.json添加prepare script
npm set-script prepare "husky install"  
npm run prepare  

添加commit hook（提交信息前运行）:  
npx husky add .husky/pre-commit "npx lint-staged"  
git add .husky/pre-commit  

添加commit-msg hook（提交成功前，验证提交信息）:
npx husky add .husky/commit-msg 'yarn commitlint --edit $1' 

```

lint-staged：lint-staged能够让lint只检测暂存区的文件

```bash
  "lint-staged": {
    "*.{js,css,md,ts,tsx}": "prettier --write"
  }
```

commitlint：代码提交信息规范

```bash
npm install --save-dev @commitlint/{config-conventional,cli}
echo "module.exports = {extends: ['@commitlint/config-conventional']}" > commitlint.config.js

yarn husky add .husky/commit-msg 'yarn commitlint --edit $1'
```


### mock