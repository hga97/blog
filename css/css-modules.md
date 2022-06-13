css的样式应用是全局性的，没有作用域可言。  
如组件和页面间或组件嵌套使用了相同的 class 属性，会发生 CSS 样式覆盖的问题。

css隔离主要有两类方案：  
**1、运行时通过命名区分**  
**2、编译时的自动切换css，添加上模块的唯一标识**  

运行时典型的方案就是BEM，通过命名规范来实现样式的隔离。  
编译时的方案有两种，一种是 scoped，另一种是 css modules  

### BEM

[BEM](https://www.bemcss.com/#) 是一种典型的 CSS 命名方法论，由 Yandex 团队在 2009 年前提出，它的核心思想是
**通过组件名的唯一性来保证选择器的唯一性，从而保证样式不会污染到组件外。**  
BEM的命名规矩很容易记：block-name__element-name--modifier-name，也就是模块名 + 元素名 + 修饰器名。

### scoped

scoped 是 vue-loader 支持的方案，它是通过编译的方式在元素上添加了 data-xxx 的属性，
然后给 css 选择器加上[data-xxx] 的属性选择器的方式实现 css 的样式隔离。

```vue
<template>
    <div class="home">
    </div>
</template>

<style scoped>
.home {
    height: 100vh;
    overflow: hidden;
    position: relative;
}
</style>
```

编译为

```html
<div data-v-3dd2e005 class="home"></div>
```

```css
.home[data-v-3dd2e005] {
    height: 100vh;
    overflow: hidden;
    position: relative;
}
```


### CSS Moudules

css-modules 是 css-loader 支持的方案，在 vue、react 中都可以用，它是通过编译的方式修改选择器名字为全局唯一的方式来实现 css 的样式隔离。

#### Vue

Vue [CSS Moudules](https://vue-loader.vuejs.org/guide/css-modules.html)

#### React

```js
<div className={appStyles.carBox}>
    我是car
</div>
```

```css
.carBox {
  position: fixed;
  left: 0;
  right: 0;
  bottom: 0;
  height: 100px;
  background-color: red;
  border-radius: 10px 10px 0 0;
  transform: translate3d(0, 100%, 0);
  transition: all 0.1s;
  z-index: 11;
}
```

编译为

```css
.App_carBox__CKfqN {
    position: fixed;
    left: 0;
    right: 0;
    bottom: 0;
    height: 100px;
    background-color: red;
    border-radius: 10px 10px 0 0;
    transform: translate3d(0, 100%, 0);
    transition: all 0.1s;
    z-index: 11;
}
```

### 覆盖隔离后的样式

#### scope

1、子组件的根元素

使用 scoped 后，父组件的样式将不会渗透到子组件中。不过一个子组件的根节点会同时受其父组件有作用域的 CSS 和子组件有作用域的 CSS 的影响。
这样设计是为了让父组件可以从布局的角度出发，调整其子组件根元素的样式。

2、深度作用选择器

如果你希望 scoped 样式中的一个选择器能够作用得“更深”，例如影响子组件，你可以使用 >>> 操作符：

```vue
<style scoped>
.a >>> .b { /* ... */ }
</style>
```

编译为

```css
.a[data-v-f3f3eg9] .b { /* ... */ }
```

有些像 Sass 之类的预处理器无法正确解析 >>>。这种情况下你可以使用 /deep/ 操作符取而代之——这是一个 >>> 的别名，同样可以正常工作。

#### React CSS Modules

```js
<div className={styles.app}>
    <Transition className={styles.carBox} />
</div>

// transition.js
<div className={`${styles.carBox} ${show ? styles.carBoxUp : ""}`} data-role="car-box">
    我是car
</div>
```

```css
.app {
  [data-role="car-box"] {
    background-color: blue;
    color: #fff;
  }
}
```

编译为

```css
.App_app__GuJBs [data-role="car-box"] {
    background-color: blue;
    color: #fff;
}

.transition_carBox__BjD8p {
    position: fixed;
    left: 0;
    right: 0;
    bottom: 0;
    height: 100px;
    background-color: red;
    border-radius: 10px 10px 0 0;
    transform: translate3d(0, 100%, 0);
    transition: all 0.1s;
    z-index: 11;
}
```


<!-- 
https://zhuanlan.zhihu.com/p/20495964?columnSlug=purerender

疑问

css隔离的组件，加了全局样式后，怎么作用？作用得到吗？

答
react: 没有用，覆盖不了。
vue: 有用，但覆盖不了原先设置的样式。

css隔离的组件，怎么更改起样式？

答
上面有说。

小程序组件的处理，以及前面的问题？

答
后面整理


less
sass
css-in-js 
Tailwind

-->

