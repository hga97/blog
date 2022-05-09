### .env文件
Dotenv 是一个零依赖模块，可将 .env 文件中的环境变量加载到 process.env 中。  
通过DefinePlugin 插件，再将这些环境变量加载到js文件中。  
.env文件在启动之后是只读的，如果想让修改生效则需重启。  

?> DefinePlugin  
DefinePlugin 允许在编译时将你代码中的变量替换为其他值或表达式。这在需要根据开发模式与生产模式进行不同的操作时，非常有用。

```js
new webpack.DefinePlugin({
  'process.env.NODE_ENV': JSON.stringify(process.env.NODE_ENV),
});
```

### fetch
用于访问和操纵 HTTP 管道的一些具体部分，例如请求和响应。它还提供了一个全局 fetch() 方法，
该方法提供了一种简单，合理的方式来跨网络异步获取资源。

支持的请求参数  
fetch() 接受第二个可选参数，一个可以控制不同配置的 init 对象（RequestInit）：  

```js
 {
    method: 'POST', // *GET, POST, PUT, DELETE, etc.
    mode: 'cors', // no-cors, *cors, same-origin
    cache: 'no-cache', // *default, no-cache, reload, force-cache, only-if-cached
    credentials: 'same-origin', // include, *same-origin, omit
    headers: {
      'Content-Type': 'application/json'
      // 'Content-Type': 'application/x-www-form-urlencoded',
    },
    redirect: 'follow', // manual, *follow, error
    referrerPolicy: 'no-referrer', // no-referrer, *no-referrer-when-downgrade, origin, origin-when-cross-origin, same-origin, strict-origin, strict-origin-when-cross-origin, unsafe-url
    body: JSON.stringify(data) // body data type must match "Content-Type" header
  }
```

响应Response  
当接收到一个代表错误的 HTTP 状态码时，从 fetch() 返回的 Promise 不会被标记为 reject，即使响应的 HTTP 状态码是 404 或 500。
相反，它会将 Promise 状态标记为 resolve （如果响应的 HTTP 状态码不在 200 - 299 的范围内，则设置 resolve 返回值的 ok 属性为 false ），
仅当网络故障时或请求被阻止时，才会标记为 reject。  

返回一个包含响应结果的 promise（一个 Response 对象），它只是一个 HTTP 响应，而不是真的 JSON。为了获取JSON的内容，
我们需要使用 json() 方法（该方法返回一个将响应 body 解析成 JSON 的 promise）

### qs
一个查询字符串解析和字符串化库，  
qs.stringify(): 将对象序列化成url的形式，以&进行拼接。  
qs.parse(): 将url解析成对象形式。  

?> 使用技巧  
如跳转url携带参数，需要校验每个对象的值是否为空，在拼接。  
将需要携带的参数，合并成一个对象，过滤空值。
再使用qs拼接参数

```js
export const isFalsy = (value: unknown) => {
  return value === 0 ? false : !value;
};

export const cleanObject = (object: Record<string, unknown>) => {
  const result = { ...object };
  for (const key in result) {
    if (isFalsy(result[key])) {
      delete result[key];
    }
  }
  return result;
};

endPoint += `?${qs.stringify(data)}`;
```

### useEffect 

在 React 组件中有两种常见副作用操作：需要清除的和不需要清除的。  

**无需清除的 effect**  
在 React 更新 DOM 之后运行一些额外的代码。比如发送网络请求，手动变更 DOM，记录日志，这些都是常见的无需清除的操作。  

useEffect 做了什么:  React 组件需要在渲染后执行某些操作。React 会保存你传递的函数（我们将它称之为 “effect”），并且在执行 DOM 更新之后调用它。  
useEffect 会在每次渲染后都执行吗？： 默认情况下，它在第一次渲染之后和每次更新之后都会执行。useEffect传入第二个可选参数, 仅在可选参数更改时更新。  

```js
// 仅执行一次
export const useMount = (callback: () => void) => {
  useEffect(() => {
    callback();
  }, []);
};
```

**需要清除**

一些副作用是需要清除的。例如订阅外部数据源。这种情况下，清除工作是非常重要的，可以防止引起内存泄露！  

为什么要在 effect 中返回一个函数？: 这是 effect 可选的清除机制。每个 effect 都可以返回一个清除函数。  
React 何时清除 effect: effect 在每次渲染的时候都会执行。这就是为什么 React 会在执行当前 effect 之前对上一个 effect 进行清除。  

```js
export const useDebounce = <V>(value: V, delay: number) => {
  // 状态（响应式：页面跟着更改）
  const [debounceValue, setDebounceValue] = useState(value);
  useEffect(() => {
    const time = setTimeout(() => {
      console.error("debounce---");
      setDebounceValue(value);
    }, delay);

    // 每次在上一个useEffect处理完以后再运行
    return () => {
      clearTimeout(time);
    };
  }, [value, delay]);
  return debounceValue;
};
```