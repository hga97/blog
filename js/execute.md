### 顺序执行？

```js
var foo = function () {
    console.log('foo1');
}

foo(); // foo1

var foo = function () {
    console.log('foo2');
}

foo(); // foo2
```

or

```js
function foo() {
    console.log('foo1');
}

foo(); // foo2

function foo() {
    console.log('foo2');
}

foo(); // foo2
```

?> JavaScript 引擎并非一行一行地分析和执行程序，而是一段一段地分析执行。当执行一段代码的时候，
会进行一个“准备工作”，比如第一个例子中的**变量提升**，和第二个例子中的**函数提升**。

一段一段 怎么划分  
遇到一段怎样的代码才会做准备工作  

### 可执行代码
可执行代码的类型：全局代码、函数代码和eval代码  

准备工作 --->>> **执行上下文**  

js引擎创建了执行上下文栈（ECS）来管理执行上下文  

模拟执行上下文栈的行为，让我们定义执行上下文栈是一个数组：  

```js
ECStack = []; 
```

js解释执行代码的时候，最先遇到的是全局代码，所以初始化的时候首先会向执行上下文栈压入一个全局执行上下文（globalContext）。
并且只有当整个应用程序结束的时候，ECStack才会被清空，所以程序结束之前，ECStack最底部永远有个globalContext。  

```js
ECStack = [ globalContext ];
```

JavaScript 遇到下面的这段代码:  

```js
function fun3() {
    console.log('fun3')
}

function fun2() {
    fun3();
}

function fun1() {
    fun2();
}

fun1();
```

当执行一个函数的时候，就会创建一个执行上下文，并且压入执行上下文栈，
当函数执行完毕的时候，就会将函数的执行上下文从栈中弹出。知道了这样的工作原理，让我们来看看如何处理上面这段代码：  

```js
// 伪代码

// fun1()
ECStack.push(<fun1> functionContext);

// fun1中竟然调用了fun2，还要创建fun2的执行上下文
ECStack.push(<fun2> functionContext);

// 擦，fun2还调用了fun3！
ECStack.push(<fun3> functionContext);

// fun3执行完毕
ECStack.pop();

// fun2执行完毕
ECStack.pop();

// fun1执行完毕
ECStack.pop();

// javascript接着执行下面的代码，但是ECStack底层永远有个globalContext
```

[在这里插入图片描述](https://img-blog.csdnimg.cn/84bb088404dd49b196d81c28ae29183b.png)

### 解答思考题

解答[作用域](js/scope.md)的思考题

```js
var scope = "global scope";
function checkscope(){
    var scope = "local scope";
    function f(){
        return scope;
    }
    return f();
}
checkscope();

ECStack = [ globalContext ];
ECStack.push(<checkscope> functionContext);
ECStack.push(<f> functionContext);
ECStack.pop(); // f执行完
ECStack.pop(); // checkscope执行完

```

```js
var scope = "global scope";
function checkscope(){
    var scope = "local scope";
    function f(){
        return scope;
    }
    return f;
}
checkscope()();

ECStack = [ globalContext ];
ECStack.push(<checkscope> functionContext);
ECStack.pop(); // checkscope执行完
ECStack.push(<f> functionContext);
ECStack.pop(); // f执行完

```