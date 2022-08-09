JavaScript代码执行一段可执行代码时，会创建对应的执行上下文。

执行上下文 --->>> 三个重要的属性  
**变量对象**(Variable object，VO)  
**作用域链**(Scope chain)  
**this**  

### 变量对象

变量对象是与**执行上下文相关的数据作用域，存储了在上下文中定义的变量和函数声明**。  
全局上下文下的变量对象和函数上下文下的变量对象。

### 全局上下文

全局对象是预定义的对象，作为js的全局函数和全局属性的占位符。通过使用全局对象，可以访问所有其他预定义的对象、函数和属性。  

在顶层js代码中，可以用关键字this引用全局对象。因为全局对象是作用域链的头，这意味着所有非限定性的变量和函数名都会作为该对象的属性来查询。  

1、可以通过this引用，在客户端js中，全局对象就是window对象  

```js
console.log(this); // window
```

2、全局对象是由obj构造函数实例化的一个对象  

```js
console.log(this instanceof Object); //true
```

3、预定义一大堆函数和属性  

```js
console.log(Math.random());
console.log(this.Math.random());
```

4、作为全局变量的宿主  

```js
var a = 1;
console.log(this.a);
```

5.客户端 JavaScript 中，全局对象有 window 属性指向自身。

```js
var a = 1;
console.log(window.a);

this.window.b = 2;
console.log(this.b);
```

**全局上下文中的变量对象就是全局对象呐！**

### 函数上下文

在函数上下文中，我们用活动对象（activation object, AO）来表示变量对象。  

活动对象和变量对象其实是一个东西，只是变量对象是规范上的或者说是引擎实现上的，不可在js环境中访问，
只有到当进入一个执行上下文中，这个执行上下文的变量才会被激活，所以才叫 activation object 呐，
而只有被激活的变量对象，也就是活动对象上的各种属性才能被访问

活动对象是在进入函数上下文时刻被创建的，它通过函数的 arguments 属性初始化。arguments 属性值是 Arguments 对象。  

### 执行过程

执行上下文的代码会分成两个阶段进行处理：分析和执行，我们也可以叫做：  
1、进入执行上下文  
2、代码执行  

##### 进入执行上下文
（不是执行的过程）
当进入执行上下文时，这时候还没有执行代码，变量对象会包括：  
1、函数的所有行参（如果是函数上下文）  
    由名称和对应值组成的一个变量对象的属性被创建  
    没有实参，属性值设为undefined

2、函数声明  
    由名称和对应值（函数对象(function-object)）组成一个变量对象的属性被创建  
    如果变量对象已经存在相同名称的属性，则完全替换这个属性   

3、变量声明  
    由名称和对应值（undefined）组成一个变量对象的属性被创建  
    如果变量名称跟已经声明的形式参数或函数相同，则变量声明不会干扰已经存在的这类属性

例子：  
```js
function foo(a) {
  var b = 2;
  function c() {}
  var d = function() {};

  b = 3;

}

foo(1);
```

进入执行上下文  

```js
AO = {
    arguments: {
        0: 1,
        length: 1
    },
    a: 1,
    b: undefined,
    c: reference to function c(){},
    d: undefined
}

```

##### 代码执行

在代码执行阶段，会顺序执行代码，根据代码，修改变量对象的值

还是上面的例子，当代码执行完后，这时候的 AO 是：

```js
AO = {
    arguments: {
        0: 1,
        length: 1
    },
    a: 1,
    b: 3,
    c: reference to function c(){},
    d: reference to FunctionExpression "d"
}
```
总结：  
1、全局上下文的变量对象初始化是全局对象  
2、函数上下文的变量对象初始化只包括Arguments对象  
3、在进入执行上下文时会给变量对象添加形参、函数声明、变量声明等初始化的属性值  
4、在代码执行阶段，会再次修改变量对象的属性值  

issues讨论：  
确实，变量提升是一个在规范中找不到的术语。变量提升被认为是思考执行上下文（特别是创建和执行阶段）在 JavaScript 中如何工作的一种方式。不过如果直接用变量对象去解释，估计给新手会增加不少的理解成本~

```js
console.log(foo); //foo函数

function foo(){
    console.log("foo");
}

var foo = 1;

console.log(foo) // 1

// 函数声明>变量声明
// 先提升为foo(), 后赋值为1
```

```js
var foo = 1;
console.log(foo);
function foo(){
  console.log("foo");
};
console.log(foo);
// 这次打印结果就是“1”；

// 分解
var foo; 
foo = 1;
console.log(foo); // 1
function foo(){ 
  console.log("foo");
};
console.log(foo) // 1 

// 函数声明>变量声明
// 先提升为foo(), 后赋值为1
```


