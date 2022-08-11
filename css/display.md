例子：购物车规格选择器弹窗，背景颜色显示和隐藏的渐变。

## 框架实现

### Vue

Vue 提供了 transition 的封装组件，在下列情形中，可以给任何元素和组件添加进入/离开过渡

- 条件渲染 (使用 v-if)
- 条件渲染 (使用 v-show)
- 动态组件
- 组件根节点

**进入/离开的过渡中,class 切换**
![在这里插入图片描述](https://img-blog.csdnimg.cn/ef062398341644f1a7258b55587752cf.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAd2VpeGluXzM5OTgxNjcx,size_20,color_FFFFFF,t_70,g_se,x_16)

```html
<div id="app">
  <div @click="isShow=!isShow">购物车</div>
  <transition name="fade">
    <div v-if="isShow" @click="isShow=false" class="mask"></div>
  </transition>
  <div class="car-box" :class="{ 'car-box-up': isShow}">我是car</div>
</div>
```

```js
var app = new Vue({
  el: "#app",
  data: {
    isShow: false,
  },
});
```

```css
.car-box {
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

.car-box-up {
  transform: translate3d(0, 0, 0);
}

.mask {
  position: fixed;
  left: 0;
  right: 0;
  bottom: 0;
  top: 0;
  background-color: rgba(7, 17, 27, 0.6);
  z-index: 1;
}

.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.5s;
}

.fade-enter, .fade-leave-to /* .fade-leave-active below version 2.1.8 */ {
  opacity: 0;
}
```

### React

react 官网，动画效果推荐了 react-transition-group、react-motion 和 react-spring 库。

使用 react-spring 来实现

```js
function App() {
  const [show, setShow] = useState(false);
  const transitions = useTransition(show, {
    from: { opacity: 0 },
    enter: { opacity: 1 },
    leave: { opacity: 0 },
    reverse: show,
    delay: 100,
  });

  return (
    <>
      <div
        onClick={() => {
          setShow(true);
        }}
      >
        购物车
      </div>
      {transitions(
        (styles, item) =>
          item && (
            <animated.div
              onClick={() => {
                setShow(false);
              }}
              className="mask"
              style={styles}
            ></animated.div>
          )
      )}
      <div className={show ? "car-box car-box-up" : "car-box"}>我是car</div>
    </>
  );
}
```

## 原生实现

### visibility

?> display 与 visibility 的区别  
visibility: 不可见的元素也会占据页面上的空间。visibility 属性值在 visible 与 hidden 的来回切换中，不会破坏元素的 transition 动画。  
display: 不占据页面空间的不可见元素。 属性会破坏 transition 动画。

**visibility 属性过渡实例**  
从显示到隐藏确实是等待了 1s，但从隐藏到显示，好像还是瞬间完成的，并没有等待 1s 的说法。

?> transition  
例如从 opacity:0 到 opacity:1 的变化，用时 1s，那么 transition 会计算在这 1s 内的每一帧画面中元素该有的 opacity 值，从而完成过渡效果。  
而从 visibility:hidden 到 visibility:visible 的过程中。因为没办法计算过渡阶段没帧的值，所以元素就直接显示出来了，但内在的过渡操作依旧在元素显示出来后显示了 1s。从 visibility:visible 到 visibility:hidden，元素在视觉上看起来等待的 1s 内，实际在内部已经在进行 transition 过渡操作，只不过还是因为没办法计算值，所以到了过渡阶段的最后一刻时，就直接将元素设置为结束状态，也就是隐藏了。

```css
.dv {
  visibility: visible;
  width: 200px;
  height: 200px;
  background-color: aqua;
  transition: all 1s;
}

.hide {
  visibility: hidden;
}
```

```html
<div class="btn">按钮</div>
<div class="dv"></div>
```

```js
const btn = document.querySelector(".btn");
const dv = document.querySelector(".dv");
btn.addEventListener("click", () => {
  if (dv.className.indexOf("hide") != -1) {
    dv.className = "dv";
  } else {
    dv.className = "dv hide";
  }
});
```

**结合 opacity**

visibility:hidden 的元素就会出现点透，点击事件会穿透 visibility:hidden 的元素，被下面的元素接收到，元素在隐藏的时候，就不会干扰到其他元素的点击事件。

```css
.dv {
  visibility: visible;
  width: 200px;
  height: 200px;
  background-color: aqua;
  transition: all 1s;
  opacity: 1;
}

.hide {
  opacity: 0;
  visibility: hidden;
}
```

### setTimeOut

visibility 有兼容问题，如果要兼容老版本的浏览器，如：ie9。可以使用 setTimeOut 来。

```html
<div class="dv" style="display: none"></div>
```

```css
.dv {
  width: 100px;
  height: 100px;
  background-color: aqua;
  transition: all 1s;
  opacity: 0;
}
```

```js
const btn = document.querySelector(".btn");
const dv = document.querySelector(".dv");
btn.addEventListener("click", () => {
  const dvDisplay = dv.style.display;
  if (dvDisplay === "none") {
    dv.style.display = "block";
    setTimeout(() => {
      dv.style.opacity = 1;
    });
  } else {
    dv.style.opacity = 0;
    setTimeout(() => {
      dv.style.display = "none";
    }, 1000);
  }
});
```

隐藏到显示：setTimeOut 一个重要功能就是延迟执行，只要将 opacity 属性的设置延迟到 display 后面执行就行了。  
显示到隐藏：隐藏时先设置 opacity，等 opacity 过渡完了，再设置 display:none。

但是这里有点不太合理，因为虽然 setTimeOut 的 delay 参数 1000ms 和 transition 时间 1s 一样大，但因为 JS 是单线程，遵循时间轮询，所以并不能保证 display 属性的设置刚好是在 opacity 过渡完了的同时执行，可能会有更多一点的延迟，这取决于过渡动画完成之刻， JS 主线程是否繁忙。

更完美的方案

### transitionend



<!-- 
https://www.infoq.cn/article/javascript-high-performance-animation-and-page-rendering/
 -->
