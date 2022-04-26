作用域是指**程序源代码中定义变量的区域。**  

作用域规定了如何查找变量，也就是确定当前执行代码对变量的访问权限。  

js采用词法作用域(lexical scoping)，也就是静态作用域。  


### 静态作用域与动态作用域

词法作用域，**函数的作用域在函数定义的时候就决定了。**

动态作用域，**函数的作用域是在函数调用的时候才决定的。**

```js
var value = 1;

function foo() {
    console.log(value); //在全局作用域中找到value为1
}

function bar() {
    var value = 2;
    foo();
}

bar(); // 1
```

### 动态作用域

``` bash
value=1
function foo () {
    echo $value;
}
function bar () {
    local value=2;
    foo;
}
bar // 2
```

### 思考题

```js
var scope = "global scope";
function checkscope(){
    var scope = "local scope";
    function f(){
        return scope;
    }
    return f();
}
checkscope(); // "local scope"
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
checkscope()(); // "local scope"
```

原因也很简单，因为JavaScript采用的是词法作用域，**函数的作用域基于函数创建的位置。**

javaScript权威指南

JavaScript 函数的执行用到了**作用域链**，这个作用域链是在函数定义的时候创建的。
嵌套的函数 f() 定义在这个作用域链里，其中的变量 scope 一定是局部变量，不管何时何地执行函数 f()，这种绑定在执行 f() 时依然有效。