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


## 原生实现

### visibility

### setTimeOut

### transitionend