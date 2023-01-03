# BEM规范

> 由于项目开发中,每个组件都是独一无二的,其名字也是独一无二的,组件内部元素的名字加上组件名,并用元素的名字座位[选择器](https://so.csdn.net/so/search?q=选择器&spm=1001.2101.3001.7020),自然组件内的样式就不会与组件外的样式冲突了,通过组件名的唯一性来保证选择器的唯一性,从而保证样式不会污染到组件外

**BEM规范，即`Block`（块）`Element`（元素）`Modifier`（修饰器)，用来规范css命名**

- **命名约定模式**

对于块，若多个单词，则用 - 连接，如search-form

```css
.block{}
.block__element{}
.block__element--modifier{}
```

- **具体例子**
  - 块即模块，如搜索表单search-form，可以看做一个块
  - 这个块内的按钮button、输入框input，为元素
  - 元素可以有多种状态，如居中按钮，即修饰

```html
<form class="search-form">
    <input class="search-form__input"/>              //元素
    <button class="search-form__button"></button>    //元素
    <button class="search-form__button--primary"></button>    //修饰
</form>
```
