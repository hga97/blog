### 实现 patch

patch 功能是把要渲染的 vdom 和已有的 dom 做下 diff，只更新需要更新的 dom，也就是按需更新。

是否要走 patch 逻辑，这里可以加一个 shouldComponentUpdate 来控制，如果 props 和 state 都没变就不用 patch 了。

```js
setState(nextState) {
    this.state = Object.assign(this.state, nextState);

    if(this.dom && this.shouldComponentUpdate(this.props, nextState)) {
        patch(this.dom, this.render());
    }
}

shouldComponentUpdate(nextProps, nextState) {
    return nextProps != this.props || nextState != this.state;
}
```

#### 文本

- 如果 vdom 不是文本节点，直接替换
- 如果 vdom 是文本节点，对比下内容，内容不一样就替换

#### 组件

加上 instance 属性，更新的时候就可以对比下 constructor 是否一样，如果一样说明是同一个组件，那 dom 是差不多的，再 patch 子元素，否则，不是同一个组件的话，直接替换。

#### 元素

- 不同类型的元素，直接替换
- 同一类型的元素，更新子节点和属性

更新子节点，能重用就重用，所以 render 的时候给每个元素加上一个标识 key。更新的时候如果找到 key 就重用，没找到就 render 一个新的。

#### 总结

patch 和 render 一样，也是递归的处理元素、组件和文本。

patch 时要对比下 dom 中的和要渲染的 vdom 的一些信息，然后决定渲染新的 dom，还是复用已有 dom，所以 render 的时候要在 dom 上记录 instance、key 等信息。

元素的子元素更新要支持 key 做标识，这样可以复用之前的元素，减少 dom 的创建。属性设置的时候 event listener 要每次删掉已有的再添加一个新的，保证只会有一个。
