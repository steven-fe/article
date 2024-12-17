# 由一个变量名引发的概念问题 —— host, domain, origin, referer 的区别

一位前端同事利用postMessage开发需求，在做code review时，发现了一个微小的细节问题。其部分代码如下：

```js
const hostUrl = new URL(targetUrl).origin;

export function setMsg(value) {
  if (!contentWindow) {
    return;
  }
  contentWindow.postMessage(
    {
      method: 'set',
      value,
    },
    hostUrl,
  );
}
```

postMessage方法的第二个参数传入变量名 `hostUrl` 。第一次看的时候，并未发现问题。但第二次再看时，引发了一个问题：为什么变量名不写成 `targetOrigin` ？`origin` 与 `host` 有什么区别？`host` 与 `domain` 有什么区别？

一系列的问题，自己也难以回答。为了避免以后再出现这种尴尬场面。于是参考了资料，整理如下。

## host

用来标记服务器的具体地址，其中包括 `域名` 和 `端口号` 。例如：

> 如果端口为协议的默认端口，则可以省略。（如 HTTP 的 80、HTTPS 的 443）

## domain

用来标记服务器域名，其中仅包括包括 `域名` 。例如：

```http
www.baidu.com
```

## origin

用来标记请求的来源，其中仅包括包括 `协议`, `域名`, `端口号`。在处理一些网络安全方面的问题时，经常会用到这个数据。例如：

```html
http://www.baidu.com:8888
```

> 如果端口为协议的默认端口，则可以省略。（如 HTTP 的 80、HTTPS 的 443）

## referer

用来标记发送请求的来源，包括url地址栏上的`所有数据`（注意，不包含锚点数据）。例如：

```html
https://www.baidu.com/s?wd=23412&rsv_spt=1&rsv_iqid=0xf8e117cb002d9442&issp=1&f=8&rsv_bp=1&rsv_idx=2&ie=utf-8&tn=baiduhome_pg&rsv_enter=0&rsv_dl=tb&rsv_sug3=6&rsv_sug1=2&rsv_sug7=101&rsv_btype=i&prefixsug=2%2526lt%253B412&rsp=0&inputT=1385&rsv_sug4=1589
```

## 总结

介绍完`host`, `domain`, `origin`, `referer`之间的区别之后，回头再看看开头介绍的postMessage传入参数的变量名应该为`targetOrigin`更合适一些。
