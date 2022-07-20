# Web Storage

cookie 为浏览器提供了存储数据的方案，但该方案有如下缺点：

1. 存储空间有限，只有 4KB；随着网页存储浏览器的数据量增加，这点存储空间捉襟见肘。
2. 浏览器发起请求时，都会携带符合要求的 cookie 数据；与接口无关的数据存储在 cookie 中，会增加每次请求的载荷。
3. cookie 初衷为存储用户的身份标识，存储其他数据，则与设置初衷背离。

为了解决 cookie 在浏览器存储数据方面的缺陷，HTML5 带来了 Web Storage 方案。

Web Storage 方案定义了两个存储对象：localStorage 和 sessionStorage。localStorage 是永久存储机制，sessionStorage 是跨会话的存储机制。

## Storage 类型

localStorage 和 sessionStorage 对象都是 Storage 构造函数的实例，都具有相同的操作属性和方法。这些方法都是同步方法，即存储任务执行完毕之后才继续执行之后的代码。

### setItem

存储键/值对；如果键存在，则覆盖值。

```ts
class Storage {
  setItem(key: string, value: string): void
}
```

### getItem

按照键返回对应值；如果键对应的值存在，则返回对应的值，否则返回`null`。

```ts
class Storage {
  getItem(key: string): string | null
}
```

### removeItem

删除键以及对应的值。

```ts
class Storage {
  removeItem(key: string): void
}
```

### clear

删除所有的数据。

```ts
class Storage {
  clear(): void
}
```

### key

获取该索引下的键名；获取不到键名时，返回`null`。

```ts
class Storage {
  key(index: number): string | null
}
```

### length

获取存储键值对的数量。

```ts
class Storage {
  length: number
}
```

## sessionStorage

sessionStorage 对象存储的数据，只存在于当前标签页，其他相同 URL 的标签页无法共享 sessionStorage 数据。

当前标签页刷新时，sessionStorage 数据仍然保留，但关闭当前标签页之后 sessionStorage 数据将会删除。

> tips: 同一个标签页下符合[同源](#同源定义)要求的 iframe 页面之间的 sessionStorage 数据是可以共享的。[试试看](https://codepen.io/chinatjc/pen/NWywoXQ)

## localStorage

- 符合[同源](#同源定义)要求的所有页面都可以共享 localStorage 数据。
- 数据不会过期，浏览器重启之后数据仍然存在。

## storage 事件

```js
window.addEventListener ('storage', event => { })
```

当`localStorage`或`sessionStorage`中的数据更新后，`storage`事件就会触发，它具有以下属性：

- key —— 发生更改的数据的 key（如果调用的是 .clear() 方法，则为 null）。
- oldValue —— 旧值（如果是新增数据，则为 null）。
- newValue —— 新值（如果是删除数据，则为 null）。
- url —— 发生数据更新的文档的 url。
- storageArea —— 发生数据更新的 localStorage 或 sessionStorage 对象。

> tips: 该事件会在所有可以访问存储对象的 window 对象上触发（导致当前存储数据改变的 window 对象除外）。

## 同源定义

如果两个 URL 的协议、域名、端口都相同，则这两个 URL 是同源。

下表给出了与 URL http://store.company.com/dir/page.html 的源进行对比的示例:

| URL                                             | 结果 | 原因                            |
| ----------------------------------------------- | ---- | ------------------------------- |
| http://store.company.com/dir2/other.html        | 同源 | 只有路径不一样                  |
| http://store.company.com/dir/inner/another.html | 同源 | 只有路径不一样                  |
| https://store.company.com/secure.html           | 失败 | 协议不同                        |
| http://store.company.com:81/dir/etc.html        | 失败 | 端口不同 （http 默认端口是 80） |
| http://news.company.com/dir/other.html          | 失败 | 域名不一样                      |

## 总结

Web 存储对象`localStorage`和`sessionStorage`允许我们在浏览器中保存键/值对。

- `key`和`value`都必须为字符串。
- 存储大小限制为 5MB+，具体取决于浏览器。
- 它们不会过期。
- 数据绑定到源（域/端口/协议）。
