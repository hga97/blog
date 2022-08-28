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

##### IntersectionObserver.relativeToViewport(Object margins)

功能：指定页面显示区域作为参照区域之一  
参数：Object margins 用来扩展（或收缩）参照节点布局区域的边界

##### IntersectionObserver IntersectionObserver.relativeTo(string selector, Object margins)

功能：使用选择器指定一个节点，作为参照区域之一。  
参数：string selector 选择器，Object margins 用来扩展（或收缩）参照节点布局区域的边界

### 吸顶组件

#### 确定临界点，设置参照区域

##### 页面显示区 = sticky 内容区域（目标节点的大小一样，单位 px） + 参照区域

当上滑到 sticky 内容区域时：刚好目标节点跟 sticky 的内容区域重合（离开参照区域），相交比例为 0，触发吸顶。（intersectionRatio 为 0）  
当下滑到 sticky 内容区域时：刚好目标节点跟参照区域相交，相交比例大于 0， 取消吸顶（intersectionRatio > 0）

#### 吸顶跟取消吸顶

吸顶： isFixed 为 true  
取消吸顶： isFixed 为 false

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

### 图片懒加载

#### 原理

参照区域设置为屏幕可视区域再往下 200px，往下一点，使得未进入可视区提前加载。当图片未进入默认展示白色背景。当图片与参照区域相交时，展示图片。

#### 代码

```wxml
<image class="good-img img-fadeIn" style="{{item.show ? 'opacity: 1' : ''}}" data-index="{{index}}" data-show="{{!!item.show}}" src="{{item.show ? item.url : ''}}" />
```

```css
.good-item {
  overflow: hidden;
  margin: 17rpx 8rpx 0;
  background-color: #fff;
  border-radius: 10px;
}

.good-img {
  width: 350rpx;
  height: 350rpx;
}

.img-fadeIn {
  opacity: 0;
  transition: opacity 0.5s ease-in;
}
```

```js
wx.request({
  url: "https://example.com/goodlist",
  success: (res) => {
    const { goodList } = res.data;
    this.setData(
      {
        goodList,
        loading: false,
      },
      () => {
        // 图片渲染完在监听
        this.IntersectionObserver = wx.createIntersectionObserver(this, {
          observeAll: true,
        });

        // 往下200px，可以使得图片为进入可视区提前加载，没有与参照区域相交的还是没显示图片的
        this.IntersectionObserver.relativeToViewport({ bottom: 200 }).observe(
          ".good-img",
          (res) => {
            const { index, show } = res.dataset;
            const { goodList } = this.data;
            // show，渲染过的图片，不再重复setDate
            if (res.intersectionRatio > 0 && !show) {
              this.setData({
                ["goodList[" + index + "]"]: {
                  ...goodList[index],
                  show: true,
                },
              });
            }
          }
        );
      }
    );
  },
});
```

### 商品详情联动

#### 原理

使用 relativeTo 设置参照区域（tabList 节点）与目标节点（商品节点、评价节点、详情节点和推荐节点）相交情况，来选中对应的 tab。

临界点：

向上滑动时，目标节点与参照区域相交，选中当前目标节点对应的 tab  
再往上滑，上一个目标节点离开参照区域，不做处理

往下滑动时，上一个目标节点进入参照区域，不做处理  
再往下滑动时，目标节点离开参照区域，选中上一个目标节点对应的 tab

不做处理的两种情况：boundingClientRect.top < 0 , 即目标节点在参照区域上方进入或离开。

#### 代码

```wxml
  <!-- 商品 -->
  <view class="good" data-target="0">
    商品
  </view>
  <!-- 评价 -->
  <view class="comment" data-target="1">
    评价
  </view>
  <!-- 推荐 -->
  <view class="guess" data-target="2">
    推荐
  </view>
  <!-- 详情 -->
  <view class="detail" data-target="3">
    详情
  </view>
  <view class="tab-list">
    <view wx:for="{{tabList}}" wx:key="index" class="tab-item {{target === item.name ? 'active' : ''}}">{{item.text}}</view>
  </view>
```

```js
const list = ["good", "comment", "guess", "detail"];
this.IntersectionObserver = wx.createIntersectionObserver(this, {
  observeAll: true,
});

this.IntersectionObserver.relativeTo(".tab-list").observe(
  ".good, .comment, .guess, .detail",
  (res) => {
    const { target } = res.dataset;

    if (res.boundingClientRect.top < 0) return;

    console.log(res);

    if (res.intersectionRatio) {
      this.setData({
        target: list[target],
      });
    } else {
      this.setData({
        target: list[target - 1],
      });
    }
  }
);
```

### 自动化埋点
