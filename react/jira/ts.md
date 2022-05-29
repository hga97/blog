### 泛型函数

函数的输出类型依赖函数的输入类型，或者两个输入的类型以某种形式相互关联。

在ts中，泛型就是被用来描述两个值之间的对应关系。
我们需要在函数签名里声明一个类型参数。

```ts
 function firstElement<Type>(arr: Type[]): Type | undefined {
     return arr[0]
 }
```

通过给函数添加一个类型参数 Type，并且在两个地方使用它，我们就在函数的输入(即数组)和函数的输出(即返回值)之间创建了一个关联。
现在当我们调用它，一个更具体的类型就会被判断出来：

```ts
// const s: string | undefined
const s = firstElement(["a", "b", "c"]);

// const n: number | undefined
const n = firstElement([1, 2, 3]);

// const u: undefined
const u = firstElement([]);
```

例子：

```ts
// const useDebounce: <V>(value: V, delay: number) => V
const useDebounce = <V>(value: V, delay: number) => {
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

```ts
// const useArray: <T>(initialArray: T[]) => {
//     value: T[];
//     setValue: React.Dispatch<React.SetStateAction<T[]>>;
//     add: (item: T) => void;
//     clear: () => void;
//     removeIndex: (index: number) => void;
// }
export const useArray = <T>(initialArray: T[]) => {
  const [value, setValue] = useState(initialArray);
  const add = (item: T) => setValue([...value, item]);
  const clear = () => setValue([]);
  const removeIndex = (index: number) => {
    const copy = [...value];
    copy.splice(index, 1);
    setValue(copy);
  };

  return { value, setValue, add, clear, removeIndex };
};
```