### 点击埋点

#### 方案

痛点：点击埋点上报，都是写在点击事件里面上报。忘记写了就没上报或者每次都复制同一段代码，埋点业务跟事件业务耦合在一起。

解决：通过拦截页面的生命周期和组件生命周期，遍历除生命周期函数和页面(组件)实例中常用的 api 函数外的函数，在其执行时，判断参数中是否有 type 点击事件类型，有则为其埋点，在节点上面设置埋点参数，获取其参数自动化上报。

#### 代码

```wxml
<view
  style="height: 100px;background-color: #0f0;"
  data-track-type="topType"
  data-track-click-name="topName"
  bindtap="toTops"
></view>
```

```js
// 生命周期、data和常见api

const filterParms = {
  data: 1,
  onLoad: 1,
  onShow: 1,
  onReady: 1,
  onPullDownRefresh: 1,
  onReachBottom: 1,
  onShareAppMessage: 1,
  onPageScroll: 1,
  onResize: 1,
  onTabItemTap: 1,
  onHide: 1,
  onUnload: 1,
}

// 获取除生命周期、data和常见api以外的函数
function getMethods(option) {
  const methods = []
  for (const m in option) {
    if (typeof option[m] === 'function' && !filterParms[m]) {
      methods.push(m)
    }
  }
  return methods
}

// 点击
function isClick(type) {
  const mpTaps = {
    tap: 1,
    longpress: 1,
    longtap: 1,
    contact: 1, // 客服会话
    getuserinfo: 1,
    getphonenumber: 1,
    error: 1, // 打开App失败
    launchapp: 1, // 打开App
    opensetting: 1
  }
  return !!mpTaps[type]
}

// 点击埋点
function clickTrack(option, method) {
  const oldFn = option[method]

  option[method] = function () {
    const res = oldFn.apply(this, arguments)

    if (isObject(arguments[0])) { // 获取函数参数
      const { dataset } = arguments[0].currentTarget || {}
      const type = arguments[0]['type'] // 事件类型
      const { trackType, ...args } = dataset || {}
      const trackContent = {}

      //正则匹配
      Object.keys(args).forEach((x) => {
        if (/^track[\w]*$/.test(x)) {
            // “隐式位置” \b，匹配这样的位置：它的前一个“显式位置”字符和后一个“显式位置”字符不全是 \w。
            const attr = x
                .replace(/^track/, '')
                .replace(/\B([A-Z])/g, '_$1')
                .toLowerCase()

            trackContent[attr] = args[x] // 保存参数
        }
     })

      if(type && isClick(type)){
        // 埋点
        console.log('点击埋点', trackContent)
      }
    }

    return res
  }
}

const _Page = Page // 保存原先Page实例

Page = functuin (option) { // 重新构造Page实例

  const methods = getMethods(option)

  if (methods.length) {
    for (let i = 0, len = methods.length; i < len; i++) {
      clickTrack(option, methods[i])
    }
  }

}


```

### 曝光埋点
