### IntersectionObserver

#### WXML 节点布局相交状态

节点布局相交状态 API（IntersectionObserver）可用于监听两个或多个组件节点在布局位置上的相交状态。
这一组 API 常常可以用于推断某些节点是否可以被用户看见、有多大比例可以被用户看见。

这一组 API 涉及的主要概念如下。

参照节点（relativeRect）：监听的参照节点，取它的布局区域作为参照区域。如果有多个参照节点，则会取它们布局区域的交集作为参照区域。
页面显示区域也可作为参照区域之一。  
目标节点（boundingClientRect）：监听的目标，默认只能是一个节点（使用 selectAll 选项时，可以同时监听多个节点）。  
相交区域（intersectionRect）：目标节点的布局区域与参照区域的相交区域。  
相交比例（intersectionRatio）：相交区域占参照区域的比例。  
阈值：相交比例如果达到阈值，则会触发监听器的回调函数。阈值可以有多个。

#### wx.createIntersectionObserver

创建并返回一个 IntersectionObserver 对象实例。在自定义组件或包含自定义组件的页面中，
应使用 this.createIntersectionObserver([options]) 来代替。

IntersectionObserver.relativeToViewport(Object margins)

功能：指定页面显示区域作为参照区域之一  
Object margins： 用来扩展（或收缩）参照节点布局区域的边界

### 吸顶组件

#### 确定临界点，设置参照区域

临界点：页面显示区 = sticky 内容区域（目标节点的大小一样，单位 px） + 参照区域  
当上划到 sticky 内容区域时：刚好目标节点跟 sticky 的内容区域重合，相交比例为 0，触发吸顶。（intersectionRatio 为 0）  
当下划到 sticky 内容区域时：在往下移动的话，刚好跟参照区域相交， 取消吸顶（intersectionRatio > 0）

#### 吸顶跟取消吸顶

吸顶： isFixed 为 true
取消吸顶： isFixed

```wxml
<view class="sticky-container" style="height: {{height}}rpx">
<!-- wrap 内容让其浮动 -->
  <view class="sticky-wrap {{isFixed ? 'sticky-wrap-fixed' : ''}}">
    <slot></slot>
  </view>
</view>
```

```css
.sticky-container {
  position: relative;
}

.sticky-wrap-fixed {
  position: fixed;
  left: 0;
  right: 0;
  top: 0;
  z-index: 1000;
}
```

#### 代码实现

```js
this.IntersectionObserver = this.createIntersectionObserver();

const { windowWidth } = this.getSysInfo();

// rpx（responsive pixel）: 可以根据屏幕宽度进行自适应。规定屏幕宽为750rpx。
// 换算成px = (windowWidth / 750) *  height（rpx）
const top = -(windowWidth / 750) * this.data.height;

this.IntersectionObserver.relativeToViewport({
  top,
}).observe(".sticky-container", (res) => {
  if (res.intersectionRatio) {
    // 取消吸顶
    this.setData({
      isFixed: false,
    });
  } else {
    // 触发吸顶
    this.setData({
      isFixed: true,
    });
  }
});
```
