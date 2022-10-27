### MobX

MobX 是一个身经百战的库，它通过运用透明的函数式响应编程使状态管理变得简单和可扩展。

### 核心理念

从概念上讲， MobX 会把你的应用程序视为一份电子表格。

![在这里插入图片描述](https://img-blog.csdnimg.cn/7204ffd1d01c4498a8226ae16479718a.png)

1、首先是应用状态（state）。就是那些呈图状结构分布、组成你应用状态模型的对象、数组、原始值和引用。这些值是你应用中的“单元格”。

2、其次是 derivations（派生）。也就是任何可以从你的应用状态中被自动计算出来的值。这些 derivations ——或者计算值——可以是简单的值，比如未完成待办的个数；也可以是复杂的值，比如你待办清单的 HTML 视觉表示。

3、Reactions (反应)与 derivations 非常相似。但主要的区别在于这些函数不会返回值，而会自动执行一些任务。通常这些任务与 I/O 有关。它们会确保 DOM 的更新或者在对的时间自动发出网络请求。

4、最后是 actions（行动）。actions 就是所有会修改状态的代码。MobX 会让所有由于你的 actions 引发的应用状态的改变都被全部 derivations 和 reactions 同步而顺畅地自动处理掉。

![在这里插入图片描述](https://img-blog.csdnimg.cn/c12274ce97ab4004ae363932652efdf9.png)

### 一个简单的待办 store

```ts
interface Todo {
  task: string;
  completed: boolean;
  assignee: { name: string } | null;
}

class TodoStore {
  todos: Todo[] = [];

  get completedTodosCount() {
    return this.todos.filter((todo) => todo.completed === true).length;
  }

  report() {
    if (this.todos.length === 0) return "<无>";

    const nextTodo = this.todos.find((todo) => todo.completed === false);

    return (
      `下一个待办："${nextTodo ? nextTodo.task : "<无>"}"。 ` +
      `进度：${this.completedTodosCount}/${this.todos.length}`
    );
  }

  addTodo(task) {
    this.todos.push({
      task: task,
      completed: false,
      assignee: null,
    });
  }
}

const todoStore = new TodoStore();
todoStore.addTodo("阅读 MobX 教程");
console.log(todoStore.report()); //下一个待办："阅读 MobX 教程"。 进度：0/1

todoStore.addTodo("试用 MobX");
console.log(todoStore.report()); //下一个待办："阅读 MobX 教程"。 进度：0/2

todoStore.todos[0].completed = true;
console.log(todoStore.report()); //下一个待办："试用 MobX"。 进度：1/2

todoStore.todos[1].task = "在自己的项目中试用 MobX";
console.log(todoStore.report()); //下一个待办："在自己的项目中试用 MobX"。 进度：1/2

todoStore.todos[0].task = "理解 MobX 教程";
console.log(todoStore.report()); //下一个待办："在自己的项目中试用 MobX"。 进度：1/2
```

到目前为止，这段代码并没有什么特别之处。但如果我们不是非得刻意调用 report，而是可以规定它必须在每次相关状态改变时都被调用呢？那样我们就不需要从代码中所有有可能对 report 产生影响的地方手动调用 report 了。我们确实想让最新的 report 被打印出来，但并不想因为要手动去组织它而费功夫。

### 变成响应式

幸运的是，这正是 MobX 能为我们做到的事情——自动执行只依赖于状态的代码。这样我们的 report 函数就可以自动更新，就像电子表格里的一个图表一样。为了做到这一点，TodoStore 就必须变成 observable，这样 MobX 就可以对所有正在进行的更改进行追踪。为了实现这一点，我们来改一下这个类。

还有，completedTodosCount 属性可以从待办清单中被自动派生出来。通过使用 observable 和 computed 注解，我们可以为一个对象引入 observable 属性。在下面的例子中，我们为了刻意地展示出注解而使用了 makeObservable，但我们也可以改用 makeAutoObservable(this) 来简化这个过程。

```ts
import { action, autorun, computed, makeObservable, observable } from "mobx";

interface Todo {
  task: string;
  completed: boolean;
  assignee: { name: string } | null;
}

class ObservableTodoStore {
  todos: Todo[] = [];
  pendingRequests = 0;

  // observable computed action 这些描述有什么用
  constructor() {
    makeObservable(this, {
      todos: observable,
      pendingRequests: observable,
      completedTodosCount: computed,
      report: computed,
      addTodo: action,
    });
    autorun(() => console.log(this.report));
  }

  get completedTodosCount() {
    return this.todos.filter((todo) => todo.completed === true).length;
  }

  get report() {
    if (this.todos.length === 0) return "<无>";
    const nextTodo = this.todos.find((todo) => todo.completed === false);
    return (
      `下一个待办："${nextTodo ? nextTodo.task : "<无>"}"。 ` +
      `进度：${this.completedTodosCount}/${this.todos.length}`
    );
  }

  addTodo(task) {
    this.todos.push({
      task: task,
      completed: false,
      assignee: null,
    });
  }
}

const observableTodoStore = new ObservableTodoStore();
observableTodoStore.addTodo("阅读 MobX 教程"); //下一个待办："阅读 MobX 教程"。 进度：0/1
observableTodoStore.addTodo("试用 MobX"); //下一个待办："阅读 MobX 教程"。 进度：0/2
observableTodoStore.todos[0].completed = true; //下一个待办："试用 MobX"。 进度：1/2
observableTodoStore.todos[1].task = "在自己的项目中试用 MobX"; //下一个待办："在自己的项目中试用 MobX"。 进度：1/2
observableTodoStore.todos[0].task = "理解 MobX 教程";
```

第五行代码最后并没有再触发一行日志。这是因为 report 实际上并没有因为第五行代码的重命名而发生改变——尽管它背后的数据变了。另一方面，更改第一个待办的名称确实更新了 report，因为 report 正使用着那个新名字。这很好地证明了 autorun 不仅监视观察着 todos 数组，还监视着待办条目中的各个属性。（why completedTodosCount 没变，没有触发更新？）

### 把 react 变成响应式

围绕着这同一个 store 构建一个响应式的用户界面了。

React 组件本身并不是响应式的。mobx-react-lite 包中的 observer 高阶组件包装器通过（简而言之）把 React 组件用 autorun 包装起来解决了这个问题。这样可以自动让组件和状态相同步。这跟 report 的做法在概念上并没有什么不同。

唯一 MobX 专用的代码是 observer 包装器。让每个组件在相关数据发生改变时单独重新渲染了。我们不用再调用状态的 useState setter 方法了，也不用去弄清楚怎样通过使用选择器或者需要手动配置的高阶组件来追踪应用状态中正确的那一部分。简而言之，所有组件都变得智能了。但他们是以一种傻瓜式、声明式的方式被定义出来的。

```tsx
import { observer } from "mobx-react-lite";

const TodoList = observer(({ store }) => {
  const onNewTodo = () => {
    store.addTodo(prompt("输入新的待办：", "请来杯咖啡"));
  };

  return (
    <div>
      {store.report}
      <ul>
        {store.todos.map((todo, idx) => (
          <TodoView todo={todo} key={idx} />
        ))}
      </ul>
      {store.pendingRequests > 0 ? <marquee>正在加载……</marquee> : null}
      <button onClick={onNewTodo}>新待办</button>
      <small>（双击待办进行编辑）</small>
      <RenderCounter />
    </div>
  );
});

const TodoView = observer(({ todo }) => {
  const onToggleCompleted = () => {
    todo.completed = !todo.completed;
  };

  const onRename = () => {
    todo.task = prompt("任务名称", todo.task) || todo.task;
  };

  return (
    <li onDoubleClick={onRename}>
      <input
        type="checkbox"
        checked={todo.completed}
        onChange={onToggleCompleted}
      />
      {todo.task}
      {todo.assignee ? <small>{todo.assignee.name}</small> : null}
      <RenderCounter />
    </li>
  );
});

ReactDOM.render(
  <TodoList store={observableTodoStore} />,
  document.getElementById("reactjs-app")
);
```

只需要更改数据，而不需要做更多的数据记录。MobX 会重新从 store 里的状态中自动派生并更新用户界面中相关的部分。

```ts
const store = observableTodoStore;
store.todos[0].completed = !store.todos[0].completed;
store.todos[1].task = "随机待办 " + Math.random();
store.todos.push({ task: "找到一块好奶酪", completed: true });
```

### 引用的使用

有了两个独立的 store。一个装着人员，一个装着待办。把人员 store 中的人赋给 assignee，使用了一个引用。这些改变会被 TodoView 自动识别出来。有了 MobX，你就不需要先对数据进行归一化处理和手写选择器来保证组件的更新。其实，数据存放在哪儿并不重要。只要对象被转化成了 observable，MobX 就能对它们进行追踪。用真正的 JavaScript 引用就行。只要它们跟一个 derivation 有关，MobX 就会自动对它们进行追踪。

```ts
const peopleStore = observable([{ name: "Michel" }, { name: "我" }]);
observableTodoStore.todos[0].assignee = peopleStore[0];
observableTodoStore.todos[1].assignee = peopleStore[1];
peopleStore[0].name = "Michel Weststrate";

<input
  onKeyUp={(event) => {
    peopleStore[1].name = event.target.value;
  }}
/>;
```

### 异步 actions

更新 store 中的 pendingRequests 属性好让 UI 反映出当前正在加载的状态。一旦加载完成，就更新 store 中的待办事项并减小 pendingRequests 计数器的数值。

```ts
observableTodoStore.pendingRequests++;
setTimeout(
  action(() => {
    observableTodoStore.addTodo("随机待办 " + Math.random());
    observableTodoStore.pendingRequests--;
  }),
  2000
);
```
