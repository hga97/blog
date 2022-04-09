例子：购物车规格选择器弹窗，背景颜色显示和隐藏的渐变。

## 框架实现

### Vue

Vue 提供了 transition 的封装组件，在下列情形中，可以给任何元素和组件添加进入/离开过渡  
 - 条件渲染 (使用 v-if)
 - 条件渲染 (使用 v-show)
 - 动态组件
 - 组件根节点
 
**进入/离开的过渡中,class切换**
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

react官网，动画效果列出react-transition-group、react-motion和react-spring等示例。

使用react-spring来实现

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

?> display与visibility的区别  
visibility: 不可见的元素也会占据页面上的空间。visibility属性值在visible 与 hidden的来回切换中，不会破坏元素的 transition 动画。  
display: 不占据页面空间的不可见元素。 属性会破坏transition动画。  

**visibility属性过渡实例**  
从显示到隐藏确实是等待了 1s，但从隐藏到显示，好像还是瞬间完成的，并没有等待1s的说法。

?> transition  
例如从 opacity:0 到 opacity:1的变化，用时1s，那么transition会计算在这1s内的每一帧画面中元素该有的 opacity值，从而完成过渡效果。  
从visibility:hidden到 visibility:visible的过程中。因为没办法计算过渡阶段没帧的值，所以元素就直接显示出来了，但内在的过渡操作依旧在元素显示出来后显示了1s   
而从 visibility:visible 到 visibility:hidden，元素在视觉上看起来等待的1s内，实际在内部已经在进行transition过渡操作，只不过还是因为没办法计算值，所以到了过渡阶段的最后一刻时，就直接将元素设置为结束状态，也就是隐藏了。

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

**结合opacity**

visibility:hidden的元素就会出现点透，点击事件会穿透 visibility:hidden的元素，被下面的元素接收到，元素在隐藏的时候，就不会干扰到其他元素的点击事件。

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
visibility有兼容问题，如果要兼容老版本的浏览器，如：ie9。可以使用setTimeOut来。



### transitionend