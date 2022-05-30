### login

```ts
const handleSubmit = (event: React.FormEvent<HTMLFormElement>) => {
    event.preventDefault();
    const username = (event.currentTarget.elements[0] as HTMLInputElement).value;
    const password = (event.currentTarget.elements[1] as HTMLInputElement).value;
    login({ username, password });
    // console.log(event.currentTarget.elements[0].value);
};
```

### jira-dev-tool

[jira-dev-tool](https://github.com/sindu12jun/jira-dev-tool)


### auth-provider

如果使用第三方登录，自己不用开发

```ts
getToken
login
register
logout
```

### Context

Context 提供了一个无需为每层组件手动添加 props，就能在组件树间进行数据传递的方法。

1、创建一个 Context 对象

```ts
const AuthContext = React.createContext<
  | {
      user: User | null;
      login: (form: AuthFrom) => Promise<void>;
      register: (form: AuthFrom) => Promise<void>;
      logout: () => Promise<void>;
    }
  | undefined
>(undefined);
```

2、Context.Provider

每个 Context 对象都会返回一个 Provider React 组件，它允许消费组件订阅 context 的变化。

Provider 接收一个 value 属性，传递给消费组件。一个 Provider 可以和多个消费组件有对应关系。

```ts
const AuthProvider = ({ children }: { children: ReactNode }) => {
  const [user, setUser] = useState<User | null>(null);
  const login = (form: AuthFrom) => auth.login(form).then(setUser);
  const register = (form: AuthFrom) => auth.register(form).then(setUser);
  const logout = () => auth.logout().then(() => setUser(null));

  useMount(() => {
    bootstrapUser().then(setUser);
  });

  return (
    <AuthContext.Provider
      children={children}
      value={{ user, login, register, logout }}
    />
  );
};

const AppProviders = ({ children }: { children: ReactNode }) => (
  <AuthProvider>{children}</AuthProvider>
);

<AppProviders>
    <App />
</AppProviders>
```

3、使用Context 

```ts
const useAuth = () => {
  const context = React.useContext(AuthContext);
  if (!context) {
    throw new Error("useAuth必须在AuthProvider中使用");
  }
  return context;
};

function App() {
  const { user } = useAuth();
  return (
    <div className="App">{user ? <Authenticated /> : <UnAuthenticated />}</div>
  );
}
```

### Utility Types、联合类型、Partial和Omit

1、联合类型

```ts
let myFavoriteNumber: string | number;
myFavoriteNumber = "seven";
myFavoriteNumber = 7;

let jackFavoriteNumber: string | number;
```

2、类型别名

```ts
// 定义一个类型，方便多次使用
type FavoriteNumber = string | number;

let roseFavoriteNumber: FavoriteNumber = "6";
```

3、interface

类型别名在很多情况下和interface互换

4、类型别名和interface的区别

```ts
// 定义联合类型，interface没法替代type
type f = string | number;

// interface 也没法实现Utility type
```

5、utility type

utility type 的用法：用泛型给它传入一个其他类型，然后utility type对这个类型进行某种操作

6、parameters

js typeof runtime
ts 静态环境 type http，提取变量


7、Partial和Omit

```ts
type Person = {
  name: string;
  age: number;
};
const xiaoMing: Partial<Person> = {}; //可传可不传
const shenMiRen: Omit<Person, "name" | "age"> = {}; //删除
```

### Partial、pick、exclue和Omit

1、Partial的实现

```ts

type Person = {
  name: string;
  age: number;
};

const xiaoMing: Partial<Person> = {}; //可传可不传
const xiaoMing: Partial<Person> = { name: '22'}; //可传可不传
const xiaoMing: Partial<Person> = { name: '22', age:1}; //可传可不传
type PersonKeys = keyof Person; // type PersonKeys = 'name' | 'age'

type Partial<T> = {
    [P in keyof T]?: T[P];
};

// keyof: 取key，联合类型

```

2、Pick的实现

```ts

type PersonOnlyName = Pick<Person, "name" | "age">; 

=> 

type PersonOnlyName = {
    name: string;
    age: number;
}

// extends: K是T的子集

type Pick<T, K extends keyof T> = {
    [P in K]: T[P];
};

```

3、Exclude的实现

```ts

type Age = Exclude<PersonKeys, "name">;

=> 

type Age = "age"

// 遍历时
// 为name：不返回
// 为age：返回age

type Exclude<T, U> = T extends U ? never : T;
// T:联合类型
// never：什么都没有 

```

4、Omit

```ts
const shenMiRen: Omit<Person, "name" | "age"> = {};
type Omit<T, K extends keyof any> = Pick<T, Exclude<keyof T, K>>;
```
