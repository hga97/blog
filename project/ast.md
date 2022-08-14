### AST 概念

1、AST 是什么

?> 抽象语法树（Abstract Syntax Tree）简称 AST，顾名思义，是⼀棵树，⽤分⽀和节点的组合来描述代码结构。

在计算机科学中，抽象语法和抽象语法树其实是源代码的抽象语法结构的树状表现形式。

常用的浏览器就是通过将 js 代码转化为抽象语法树来进行下一步的分析等其他操作。

2、AST 能做什么

代码语法的检查  
代码风格的检查  
代码的格式化  
代码的高亮  
代码错误提示  
代码自动补全  
...

### AST 三板斧

生成 AST  
遍历和更新 AST  
将 AST 重新生成源码

[AST Explorer](https://astexplorer.net/)

![在这里插入图片描述](https://img-blog.csdnimg.cn/2fed099e1b8047dc9c3cc98dca8fd651.png)

### AST 实战

1、使用 AST 使用一个 babel 插件，过滤 debugger

Transform 选择 babelv7

![在这里插入图片描述](https://img-blog.csdnimg.cn/2eaa6d4335bd4cc29a1da7cacb6f9d65.png)

2、把项目里的 var 全部替换成 let

Transform 选择 jscodeshift

jscodeshift 是一个工具包，用于在多个 JavaScript 或 TypeScript 文件上运行 codemods，它是：

- 一个运行器，它为传递给它的每个文件执行提供的转换。它还输出已（未）转换的文件数量统计信息；
- recast 的包装器，提供不同的 API。recast 是一个 AST 到 AST 的尽量保留原始代码的风格转换工具。

![在这里插入图片描述](https://img-blog.csdnimg.cn/3358982f39e24a9492205124986c4f47.png)

3、使用 AST 实现一个 Eslint 插件, 检查到使用 console 的时候就报错

babel-eslint 模版

![在这里插入图片描述](https://img-blog.csdnimg.cn/1812fc58c5c84252b1ca9828ffa8e109.png)