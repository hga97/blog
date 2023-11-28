开发中遇到的问题点：

- 嵌套和组织组件
- hook 闭包问题（场景：变量存放函数里面，函数没重新执行，缓存初始值。）

### 代码里面做 if 判断

```tsx
let content;
if (isLoggedIn) {
  content = <AdminPanel />;
} else {
  content = <LoginForm />;
}

return <div>{content}</div>;

// 等价：

return <div>{isLoggedIn ? <AdminPanel /> : <LoginForm />}</div>;
```

### 嵌套和组织组件

永远要将组件定义在最上层并且不要把它们的定义嵌套起来。组件更新，内部声明组件的 state 会被重置。

### 变量存放函数里面，函数没重新执行，缓存初始值

```tsx
import { Upload, message } from 'antd';
import { useState } from 'react';

function MyComponent() {
  const [list, setList] = useState([]);

  function beforeUpload(file) {
    // list 拿到的都是初始化的值
    setList([...list, file]);
  }

  return (
    <div>
      {list.map((item) => (
        <div key={item.id}>{item.name}</div>
      ))}
      <Upload beforeUpload={beforeUpload}>
        <button>选择文件</button>
      </Upload>
    </div>
  );
}

```

**在beforeUpload函数中，无法直接访问到组件中最新的list状态。这是因为beforeUpload函数是在上传文件之前被调用的，而不是在组件渲染过程中。**