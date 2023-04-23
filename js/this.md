### Types

esm 的类型分为语言类型和规范类型。

语言类型：undefined、null、boolean、string、number 和 object

规范类型: 用来算法描述 esm 语言结构和 esm 语言类型的，描述语言底层行为逻辑。

规范类型包括：Reference（引用）、List（列表）、Completion（完结）、Property Descriptor（属性描述式）、Property Identifier（属性标示）、Lexical Environment（词法环境） 和 Environment Record（环境记录）。

### Reference

reference: 只存在于规范里的抽象类型，是为了更好地描述语言的底层行为逻辑才存在的，但并不存在于实际的 js 代码中。

#### reference 的构成

- base value：它的值只可能是 undefined、an Object、a Boolean、a String、a Number 和 an environment record 其中的一种。
- referenced name：属性的名称。
- strict reference

例 1:

```js
var foo = 1;

// 对应的ref
var fooRef = {
  base: EnvironmentRecord,
  name: "foo",
  strict: false,
};
```

例 2:

```js
var foo = {
  bar: function () {
    return this;
  },
};

foo.bar(); // foo

// 对应的ref
var barRef = {
  base: foo,
  propertyName: "foo",
  strict: false,
};
```

#### 获取 reference

规范还提供了获取 reference 组成部分的方法，比如 GetBase 和 isPropertyReference。

1、GetBase

返回 reference 的 base value。

2、IsPropertyReference

如果 base value 是一个对象，就返回 true。

### GetValue

用于从 reference 类型获取对应值的方法

模拟 GetValue 的用法

```js
var foo = 1;

var fooRef = {
  base: EnvironmentRecord,
  name: "foo",
  strict: false,
};

GetValue(fooRef);
```

调用 GetValue，返回的将是具体的值，而不再是一个 Reference

### 如何确定 this 的值

函数调用，如何确定 this 的值

1、计算 MemberExpression 的结果赋值给 ref  
2、判断 ref 是不是一个 reference 类型

- 如果 ref 是 reference，并且 IsPropertyReference(ref) 是 true，那么 this 的值为 GetBase(ref)
- 如果 ref 是 reference，并且 base value 值是 Environment Record, 那么 this 的值为 ImplicitThisValue(ref)
- 如果 ref 不是 reference，那么 this 的值为 undefined

### 具体分析

#### 计算 MemberExpression 的结果赋值给 ref

MemberExpression：

- PrimaryExpression：原始表达式
- FunctionExpression：函数定义表达式
- MemberExpression[Expression]：属性访问表达式
- MemberExpression.IdentifierName：属性访问表达式
- new MemberExpression Arguments：对象创建表达式

例子：

```js
function foo() {
  console.log(this);
}

foo(); // MemberExpression 是 foo
```

```js
function foo() {
  return function () {
    console.log(this);
  };
}

foo()(); // MemberExpression 是 foo()
```

```js
var foo = {
  bar: function () {
    return this;
  },
};

foo.bar(); // MemberExpression 是 foo.bar
```

简单理解 MemberExpression 其实就是()左边的部分。

#### 判断 ref 是不是一个 reference 类型

关键就在于看规范是如何处理各种 MemberExpression，返回的结果是不是一个 Reference 类型。

例子 1

```js
var value = 1;

var foo = {
  value: 2,
  bar: function () {
    return this.value;
  },
};

//示例1
console.log(foo.bar());
//示例2
console.log(foo.bar());
//示例3
console.log((foo.bar = foo.bar)());
//示例4
console.log((false || foo.bar)());
//示例5
console.log((foo.bar, foo.bar)());
```

1、foo.bar()

11.2.1 Property Accessors， 得知该表达式返回一个 reference 类型

```js
var ref = {
  base: foo,
  name: "bar",
  strict: false,
};
```

base value 为 foo，是一个对象，所以 IsPropertyReference(ref) 结果为 true。

```js
this = GetBase(ref)
```

GetBase 也已经铺垫了，获得 base value 值，这个例子中就是 foo，所以 this 的值就是 foo ，示例 1 的结果就是 2！

2、(foo.bar)()

查看规范 11.1.6 The Grouping Operator，实际上 () 并没有对 MemberExpression 进行计算，所以其实跟示例 1 的结果是一样的。

3、(foo.bar = foo.bar)()

有赋值操作符，查看规范 11.13.1 Simple Assignment ( = )，因为使用了 GetValue，所以返回的值不是 reference 类型。this 为 undefined， 非严格模式下，this 的值为 undefined 的时候，其值会被隐式转换为全局对象。

4、(false || foo.bar)()

逻辑与算法，查看规范 11.11 Binary Logical Operators，因为使用了 GetValue，所以返回的不是 Reference 类型，this 为 undefined。

5、(foo.bar, foo.bar)()

逗号操作符，查看规范 11.14 Comma Operator ( , )，因为使用了 GetValue，所以返回的不是 Reference 类型，this 为 undefined。

例子 2

```js
function foo() {
  console.log(this);
}

foo();
```

MemberExpression 是 foo，解析标识符，查看规范 10.3.1 Identifier Resolution，会返回一个 Reference 类型的值：

```js
var fooReference = {
  base: EnvironmentRecord,
  name: "foo",
  strict: false,
};
```

base value 是 EnvironmentRecord，IsPropertyReference(ref) 的结果为 false.

base value 正是 Environment Record，所以会调用 ImplicitThisValue(ref)

查看规范 10.2.1.1.6，ImplicitThisValue 方法的介绍：该函数始终返回 undefined。

所以最后 this 的值就是 undefined。

### 总结

#### 全局执行上下文中的 this

window 对象

#### 函数执行上下文中的 this

- call、apply、bind：传入的对象
- 通过对象调用，指向对象本身
- 箭头函数的 this 就是它外层函数的 this

#### 通过构造函数中设置

new 关键字构建一个新对象，并且构造函数中的 this 就是新对象本身。
