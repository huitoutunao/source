# 认识 JSBridge

## 什么是 JSBridge

JSBridge 是以 JavaScript 实现的方法，它的作用是连接着 Native 和 H5 两端，建立双向通信的桥梁。即 APP 内有利于 H5 调用 Native 的能力，例如：扫码、拍照、查看相册等等。也方便了 Native 调用 H5 中的 JavaScript 方法。

### Native 与 H5 的比较

| 属性 | Native | H5 |
| -- | -- | -- |
| 稳定性 | 使用原生内核，更加稳定 | 调用系统浏览器内核，稳定性较差 |
| 灵活性 | 每次迭代都要平台审核，上线速度受限 | 上线灵活 |
| 流畅度 | 加载速度快且流畅 | 偶尔加载慢，有卡顿的效果 |
| 网速 | 较小 | 较大 |
| 用户体验 | 原生系统 api 丰富，能实现的功能较多，体验较好 | 功能受浏览器限制，体验有时较差 |
| 可移植性 | 对于 iOS 和 Android 需要维护两套代码 | 兼容跨平台跨系统，如 PC 与 移动端，iOS 与 Android |

## JSBridge 的通信原理简介

### JavaScript 调用 Native

#### 拦截 `URL Scheme`

Android 和 iOS 内置方法支持拦截 `URL Scheme` 并解析 scheme 来决定是否进行对应的原生代码逻辑处理。

优点是不存在漏洞问题、使用灵活，可以实现 H5 和 Native 页面的无缝切换。

缺点是使用 `iframe.src` 来发送 `URL Scheme` 需要对 URL 的长度作控制，使用复杂，速度较慢。

#### 重写 prompt

一般会通过修改浏览器的部分 Window 对象的方法来完成操作。主要是拦截 `alert`、`confirm`、`prompt`、`console.log` 四个方法，分别被 Webview 的 `onJsAlert`、`onJsConfirm`、`onConsoleMessage`、`onJsPrompt` 监听。

使用该方式时，可以与 Android 和 iOS 约定好使用传参的格式，这样 H5 可以无需识别客户端，传入不同参数直接调用 Native 即可。剩下的交给客户端自己去拦截相同的方法，识别相同的参数，进行自己的处理逻辑即可实现多端表现一致。
```js
alert('qrcode', callback())
```

#### 注入 API

基于 Webview 提供的能力，我们可以向 Window 上注入对象或方法。JS 通过这个对象或方法进行调用时，执行对应的逻辑操作，可以直接调用 Native 的方法。使用该方式时，JS 需要等到 Native 执行完对应的逻辑后才能进行回调里面的操作。

例如：
```js
window.NativeApi.share(xxx)
```

###  Native 调用 JavaScript

只要 H5 将 JS 方法暴露在 Window 上给 Native 调用即可。

## JSBridge 的使用

例如：Android 使用第三方库[JsBridge](https://github.com/lzyzsd/JsBridge)，iOS 使用第三方库[WebViewJavascriptBridge](https://github.com/marcuswestin/WebViewJavascriptBridge)。

可以做如下封装：
```js
/**
 * Android https://github.com/lzyzsd/JsBridge
 * IOS https://github.com/marcuswestin/WebViewJavascriptBridge
 * h5 与客户端通信协议
 *
 * 示例 1
 * JSBridge.h5CallNative(
 *    'fnName'
      {
        param: '0',
      },
      function (res) {
        console.log(res)
      }
    )
 * 示例 2
   JSBridge.nativeCallH5(
     'fnName',
     function (res) {
       console.log(res)
     }
   )
 */
export const JSBridge = {
  setupWebViewJavascriptBridge(callback) {
    try {
      if (window.WebViewJavascriptBridge) {
        return callback(WebViewJavascriptBridge)
      }
      if (window.WVJBCallbacks) {
        return window.WVJBCallbacks.push(callback)
      }
      window.WVJBCallbacks = [callback]
      const WVJBIframe = document.createElement('iframe')
      WVJBIframe.style.display = 'none'
      WVJBIframe.src = 'https://__bridge_loaded__'
      document.documentElement.appendChild(WVJBIframe)
      setTimeout(function () {
        document.documentElement.removeChild(WVJBIframe)
      }, 0)
    } catch (e) {
      // 处理异常
      console.log(e)
    }
  },

  /**
   * h5 调用客户端
   * @param {String} fnInNative 调用协议的方法名
   * @param {Object} param 调用协议的参数
   * @param {Function} callback 接收客户端回调函数
   */
  h5CallNative(fnInNative, param, callback) {
    this.setupWebViewJavascriptBridge(function (bridge) {
      bridge.callHandler(fnInNative, param, function (responseData) {
        if (callback) callback(responseData)
      })
    })
  },

  /**
   * 客户端调用 h5 Js 方法
   * @param {String} fnInJs 注册方法名
   * @param {Function} callback 接收客户端回调函数
   */
  nativeCallH5(fnInJs, callback) {
    this.setupWebViewJavascriptBridge(function (bridge) {
      bridge.registerHandler(fnInJs, function (responseData) {
        if (callback) callback(responseData)
      })
    })
  },
}
```

## 结语

通过这篇文章，可以简单了解 JSBridge 的概念、作用和部分原理，以及第三方库的使用。希望这篇文章能够帮助到你。

## 参考文献

[小白必看，JSBridge 初探](https://www.zoo.team/article/jsbridge)

