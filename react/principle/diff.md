### fiber 架构

在 16 之前，React 是直接递归渲染 vdom 的，setState 会触发重新渲染，对比渲染出的新旧 vdom，对差异部分进行 dom 操作。

在 16 之后，为了优化性能，会先把 vdom 转换成 fiber，也就是从树转换成链表，然后再渲染。整体渲染流程分成了两个阶段：

- render 阶段：从 vdom 转换 fiber，并且对需要 dom 操作的节点打上 effectTag 的标记
- commit 阶段：对有 effectTag 标记的 fiber 节点进行 dom 操作，并执行所有的 effect 副作用函数

diff 算法作用在 reconcile 阶段：

第一次渲染不需要 diff，直接 vdom 转 fiber。

再次渲染的时候，会产生新的 vdom，这时候要和之前的 fiber 做下对比，决定怎么产生新的 fiber，对可复用的节点打上修改的标记，剩余的旧节点打上删除标记，新节点打上新增标记。

### React 的 diff 算法

因为 dom 创建的性能成本很高，如果不做 dom 的复用，那前端框架的性能就太差了。

diff 算法的目的就是对比两次渲染结果，找到可复用的部分，然后剩下的该删除删除，该新增新增。

React 的 diff 算法是分成两次遍历的：

第一轮遍历，一一对比 vdom 和老的 fiber，如果可以复用就处理下一个节点，否则就结束遍历。

如果所有新的 vdom 处理完了，那就把剩下的老 fiber 节点删掉就行。

如果还有 vdom 没处理，那就进行第二次遍历：

第二轮遍历，把剩下的老 fiber 放到 map 里，遍历剩下的 vdom，从 map 里查找，如果找到了，就移动过来。

第二轮遍历完了之后，把剩余的老 fiber 删掉，剩余的 vdom 新增。
