# CSS变量

> 是的，CSS变量（也称为CSS自定义属性，Custom Properties）**存在作用域（Scope）**，它们的作用域遵循**层级继承**规则。

## **作用域规则**

1. 全局作用域

如果CSS变量定义在`:root`伪类中，它就具有全局作用域，可以被整个页面的所有元素访问：

```css
:root {
  --main-color: #3498db;
}
body {
  background-color: var(--main-color);
}
```

`--main-color`变量定义在`:root`，因此**任何元素都可以使用它**。

2. 局部作用域

CSS变量可以在特定的**选择器（元素、类、ID）**内定义，只对该选择器及其子元素生效：

*如果是两个定义了CSS变量的类作用在同一个元素上，那他们之间也可以共享这些变量*

```css
.container {
  --text-color: red;
}
.container p {
  color: var(--text-color);
}
.other {
  color: var(--text-color); /* 无效，因变量未继承 */
}
```

`--text-color`仅在`.container`内部可用，`.other`不能访问它。

3. 作用域继承

CSS变量**默认会被子元素继承**，但不能跨兄弟元素：

```css
.parent {
  --size: 20px;
}
.child {
  font-size: var(--size); /* ✅ 子元素可以访问 */
}
.sibling {
  font-size: var(--size); /* ❌ 兄弟元素无法访问 */
}
```

## 作用域影响的特性

**作用范围**：CSS变量的作用范围**只影响自身及子元素**，不会影响其他无关的元素。

**覆盖规则**：局部作用域的变量会覆盖全局作用域的变量：

```css
:root {
  --button-bg: blue;
}
.custom-button {
  --button-bg: red; /* 局部变量覆盖全局变量 */
  background-color: var(--button-bg);
}
```

`.custom-button`里的`--button-bg`覆盖了`:root`中的值，因此`background-color`变为 `red`。

## Fallback

如果在某个作用域内**未定义变量**，但试图使用它，会导致CSS解析失败：

```css
p {
  color: var(--undefined-color, black); /* Fallback 默认值 */
}
```

这里`--undefined-color`未定义，所以`p`的`color`变为`black`（使用了`var()`的**回退值**机制）。

## CSS变量的巧用

下面div中的两个滤镜不会自动合并，而是会相互发生覆盖。

```css
.blur-sm {
  filter: blur(4px);
}
.grayscale {
  filter: grayscale(100%);
}
```

```html
<div class="blur-sm grayscale">内容</div>
```

因为CSS中的`filter`属性是一个**单独的样式规则**，当多个类定义`filter`时，**后应用的规则会覆盖前面的规则**，而不是合并它们的值。

那如何使其合并呢？

如果希望`blur-sm`和`grayscale`仍然可以独立使用，并且在**两者同时存在时自动合并`filter`**，可以使用下面的方法：

```css
:root {
  --tw-blur:  ;
  --tw-grayscale:  ;
}
.blur-sm {
  --tw-blur: blur(4px);
  filter: var(--tw-blur) var(--tw-grayscale);
}
.grayscale {
  --tw-grayscale: grayscale(100%);
  filter: var(--tw-blur) var(--tw-grayscale);
}
```

这样`filter`的最终值就会根据变量动态调整：

- **当`.blur-sm`和`.grayscale`同时作用时**，它们的`filter`组合在一起，因为两个类的变量两者可以通用，因为虽然filter属性会互相覆盖，但是没有被覆盖掉的filter应用了两个滤镜

  ```css
  filter: blur(4px) grayscale(100%);
  ```

- 当`.grayscale`单独使用时，由于内容定义了局部CSS变量，因此仍然可以单独生效
- 但是要注意的是，全局默认变量必须被定义：`--tw-blur:  ;`，因为如果没有他的话，`.grayscale`单独使用时，就会因为`--tw-blur`变量的缺失导致filter属性解析错误
- 而如果`.grayscale`单独使用时，并且`--tw-blur`是一个托底的空串，他就会被正确的解析为正确的单个滤镜