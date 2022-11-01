### SWR

“SWR” 这个名字来自于 stale-while-revalidate：一种由 HTTP RFC 5861 推广的 HTTP 缓存失效策略。这种策略首先从缓存中返回数据（过期的），同时发送 fetch 请求（重新验证），最后得到最新数据。

### api

const { data, error, isValidating, mutate } = useSWR(key, fetcher, options)

#### 参数

key： string ｜ function ｜ array ｜ null，请求的唯一 key  
fetcher：一个请求数据的 promise 返回函数  
options: 改 swr hook 的选项对象

#### 返回值

data：通过 fetcher 用给定的 key 获取的数据（如未完全加载，返回 undefined）  
error： fetcher 抛出的错误（或者是 undefined）  
isVaildating：是否有请求或重新验证加载  
mutate：更改缓存数据的函数

#### 选项

...