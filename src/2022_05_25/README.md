# postMessage

postMessage 是 html5 引入的 API，postMessage 方法允许来自`不同源`的脚本采用异步方式进行通信，其实`同源不同页面`的脚本也可以采用 postMessage 方法进行通信。

## 介绍

### 发送数据

需要在接收数据窗口的全局对象下调用该方法。

```js
targetWindow.postMessage(message, targetOrigin, [transfer])
```

- targetWindow：目标窗口的`全局对象`引用，比如 iframe 的 contentWindow 属性、执行 window.open 返回的窗口对象、或者是命名过或数值索引的 window.frames。

- message：将要发送到`目标窗口`的数据，可以是任何类型的数据。message 数据会被[结构化克隆算法](https://developer.mozilla.org/en-US/docs/web/api/web_workers_api/structured_clone_algorithm)序列化，这意味着可以不受什么限制的将数据对象安全的传送给目标窗口而无需自己序列化。

- targetOrigin：指定目标窗口的 origin ，只有目标窗口的 origin 与 targetOrigin 对应，目标窗口才可以接收到数据。

  - \*：表示可以发送数据到任何 origin 的窗口，但通常处于安全性考虑不建议这么做。
  - \/：表示可以发送给当前窗口的同源窗口。

- transfer：可选，是一串和 message 同时传递的 Transferable 对象。这些对象的所有权将被转移给消息的接收方，而发送一方将不再保有所有权。

### 接收数据

接收数据的窗口，监听 window 对象的 message 事件，从 event 对象里获取相关数据。

```js
window.addEventListener('message', (event) => {
  const { data, origin, source } = event
  /* some code */
})
```

- data：从其他 window 中传递过来的数据，`该数据为message数据的克隆版本，而非message数据本身`。

- origin：发送窗口的 origin，一个由“协议“、“://“、“域名"、“ : 端口号”拼接而成的字符串。

- source：对发送消息的窗口对象的引用，可以使用此来在具有不同 origin 的两个窗口之间建立双向通信。

## 安全

发送数据的窗口，需要指定精确的目标窗口 origin ，而不是 \* 。

接收数据的窗口，需要通过 event.origin 判断发送数据窗口的身份。

## 实例

[两个窗口之间的通信](https://codepen.io/chinatjc/pen/VwQrYMp)
