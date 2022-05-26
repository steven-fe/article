# cookie

cookie 是存储在浏览器中的字符串数据，它是 HTTP 协议的一部分。

cookie 通常是由 Web 服务器使用响应头字段 Set-Cookie 设置的。然后浏览器将符合要求的 cookie 数据自动添加到请求头字段 Cookie 中。

cookie 常见的使用场景就是身份验证：

1. 登录后，服务器在响应中使用 Set-Cookie HTTP-header 来设置具有唯一“会话标识符（session identifier）“的 cookie 数据。

2. 下次当请求被发送到同一个域时，浏览器会使用 Cookie HTTP-header 通过网络发送 cookie 数据。

3. 所以服务器就会知道用户的身份。

除此之外，在浏览器中，还可以使用 document.cookie 读取或设置 cookie 数据。

## 读取 cookie

document.cookie 返回包含页面中所有有效 cookie 的字符串（根据域、路径、过期时间和安全设置），以分号分割，如下面的例子所示：

<!-- prettier-ignore-start -->
```js
name1=value1;name2=value2;name3=value3
```
<!-- prettier-ignore-end -->

所有名和值都是 URL 编码的，因此必须使用 decodeURIComponent() 解码。

### 辅助函数：

```js
class CookieUtil {
  static get(name) {
    let cookieName = `${encodeURIComponent(name)}=`,
      cookieStart = document.cookie.indexOf(cookieName),
      cookieValue = null

    if (cookieStart > -1) {
      let cookieEnd = document.cookie.indexOf(';', cookieStart)
      if (cookieEnd == -1) {
        cookieEnd = document.cookie.length
      }
      cookieValue = decodeURIComponent(
        document.cookie.substring(cookieStart + cookieName.length, cookieEnd)
      )
    }

    return cookieValue
  }
}
```

## 设置 cookie

通过 document.cookie 属性设置新的 cookie 字符串。这个字符串在被解析后会添加到原有的 cookie 中。设置 document.cookie 不会覆盖之前存在的任何 cookie ，除非设置了已有的 cookie 。设置 cookie 的格式如下：

<!-- prettier-ignore-start -->
```js
name=value; expires=expiration_time; path=domain_path; domain=domain_name; secure

// or

name=value; max-age=max_age_time; path=domain_path; domain=domain_name; secure
```
<!-- prettier-ignore-end -->

在所有这些参数中，只有 cookie 的 name 和 value 是必须的，且 name 和 value 必须使用 encodeURIComponent()进行编码。

### 可选字段

#### expires，max-age

默认情况下，如果一个 cookie 没有设置这两个参数中的任何一个，那么在关闭浏览器之后，该 cookie 数据就会被删除。此类的 cookie 被称为“session cookie”。

为了控制 cookie 的有效期，需要设置 `expires` 或 `max-age` 。

1. expires：定义 cookie 过期的时间，时间采用 GMT 格式（date.toGMTString()）。
2. max-age：定义 cookie 有效的时间，纯数字，单位为秒。

#### path

path 必须是绝对路径。它使得该路径下的页面都可以访问该 cookie。默认为当前路径。

如果一个 cookie 带有 path=/admin 设置，那么该 cookie 在 /admin 和 /admin/something 下都是可见的，但是在 /home 或 /adminpage 下不可见。

通常，我们应该将 path 设置为根目录：`path=/`，以使 cookie 对此网站的所有页面可见。

> tips：设置 cookie 的 path 时，可以是任意绝对路径，不受当前页面 URL 路径的影响，且都有效。

#### domain

cookie 有效的域。（域名不包含`协议`和`端口`）

发送到这个域的所有请求都会包含对应的 cookie。这个值可能包含子域（如www.wrox.com），也可以不包含（如.wrox.com 表示对 wrox.com 的所有子域都有效）。如果不明确设置，则默认为设置 cookie 的当前页面的域。

> tips：设置 cookie 的 path 时，必须符合当前页面 URL 路径的层级关系，才有效；当前页面 URL - `http://login.baidu.com`，可以设置`domain`为`login.baidu.com`或`baidu.com`，但设置`domain`为`www.baidu.com`或`www.sina.com.cn`无效。

#### secure

默认情况下，cookie 是不会区分协议的，http 协议和 https 协议都可以使用 cookie 数据。

当设置 cookie 的数据中存在`secure`标识时，则只有 https 协议可以使用该条 cookie 数据。

### 辅助函数

```js
class CookieUtil {
  static set(name, value, expiresOrMaxAge, path, domain, secure) {
    let cookieText = `${encodeURIComponent(name)}=${encodeURIComponent(value)}`

    if (expiresOrMaxAge instanceof Date) {
      cookieText += `; expires=${expiresOrMaxAge.toGMTString()}`
    } else if (typeof expiresOrMaxAge === 'number') {
      cookieText += `; max-age=${expiresOrMaxAge}`
    }

    if (path) cookieText += `; path=${path}`

    if (domain) cookieText += `; domain=${domain}`

    if (secure) cookieText += '; secure'

    document.cookie = cookieText
  }
}
```

## 删除 cookie

没有直接删除已有 cookie 的方法。为此，需要再次设置同名 cookie（包括相同路径、域和安全选项），但要将其过期时间设置为某个过去的时间。CookieUtil.unset()方法实现了这些处理。这个方法接收 4 个参数：要删除 cookie 的名称、可选的路径、可选的域和可选的安全标志。

```js
class CookieUtil {
  static unset(name, path, domain, secure) {
    CookieUtil.set(name, '', new Date(0), path, domain, secure)
  }
}
```

## 练习

[动手试试看](https://jwzz1.gitee.io/learngit/cookie.html)

## 总结

`document.cookie`提供了对`cookie`的访问。

- 写入操作只会修改其中提到的 cookie。
- name/value 必须被编码。
- 一个 cookie 最大不能超过 4KB。每个域下最多允许有 20+ 个左右的 cookie（具体取决于浏览器）。

Cookie 选项：

- expires 或 max-age 设定了 cookie 过期时间。如果没有设置，则当浏览器关闭时 cookie 就会失效。
- path=/，默认为当前路径，使 cookie 仅在该路径下可见。
- domain=site.com，默认 cookie 仅在当前域下可见。如果显式地设置了域，可以使 cookie 在子域下也可见。
- secure 使 cookie 仅在 HTTPS 下有效。

大部分场景下，cookie 存储用户的身份信息，由后端接口进行读写，前端页面很少操作 cookie 数据，但偶尔也会读写 cookie。为此写一篇文章，详细记录 cookie 在浏览器的使用规范。
