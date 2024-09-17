# 使用 css 自定义属性

## 定义

前缀 -- 的属性名，比如 --example--name，表示的是带有值的自定义属性。

```css
:root {
  --main-bg-color: blue;
}
```

“css 自定义属性”，其作用域为当前元素以及子元素。

```css
/**
 * main
 *   > div
 *     > p
 */

:root {
  --main-color: pink;
}

main {
  --main-color: red;
  color: var(--main-color); /* red, not pink */
}

div {
  color: var(--main-color); /* red, not pink */
}

p {
  color: var(--main-color); /* red, not pink */
}
```

## 使用

`var()` CSS 函数可以插入一个自定义属性（有时也被称为“CSS 变量”）的值，用来代替非自定义属性中值的任何部分。

```css
:root {
  --main-color: pink;
}

main {
  color: var(--main-color); /* pink */
}
```

具体的用法：`var(custom-property-name[, fallback-value])`

> 从第一个逗号到 var 函数的“)”为止，都是 fallback-value 的值。

> 当无法找到 custom-property-name 时，则使用 fallback-value。

### 如果 var 的 custom-property-name 属性值无效，如何处理：

1. 检查属性 color 是否为继承属性。是，但是 \<p> 没有任何父元素定义了 color 属性。转到下一步。

2. 将该值设置为它的默认初始值，比如 black。

```css
/* 1. case */
div {
  --main-color: yellow;
  color: red;
}

p {
  --main-color: 16px;
  color: blue;
  color: var(--main-color, green); /* red */
}

/* 2. case */
div {
  --main-color: yellow;
  /* color: red; */
}

p {
  --main-color: 16px;
  color: blue;
  color: var(--main-color, green); /* black(initial value) */
}
```

### 通过 javascript 获取或修改 css 自定义属性，与操作普通的 css 属性是一样的：

```js
// 获取 Dom 节点 style 属性的 CSS 变量
element.style.getPropertyValue('--my-var')
// 修改 Dom 节点 style 属性的 CSS 变量
element.style.setProperty('--my-var', jsVar + 4)

// 获取 Dom 节点上的 CSS 变量
getComputedStyle(element).getPropertyValue('--my-var')
```
