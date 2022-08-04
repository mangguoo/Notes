# Window.getComputedStyle()

> `Window.getComputedStyle()`方法返回一个对象，该对象在应用活动样式表并解析这些值可能包含的任何基本计算后报告元素的所有 CSS 属性的值。 私有的 CSS 属性值可以通过对象提供的 API 或通过简单地使用 CSS 属性名称进行索引来访问。

element

- 用于获取计算样式的Element。

pseudoElt 可选

- 指定一个要匹配的伪元素的字符串。必须对普通元素省略（或`null`）。

返回的`style`是一个实时的 `CSSStyleDeclaration`对象，当元素的样式更改时，它会自动更新本身。

``` js
let elem1 = document.getElementById("elemId");
let style = window.getComputedStyle(elem1, null);

// 它等价于
// let style = document.defaultView.getComputedStyle(elem1, null);
```

``` js
<style>
 #elem-container{
   position: absolute;
   left:     100px;
   top:      200px;
   height:   100px;
 }
</style>

<div id="elem-container">dummy</div>
<div id="output"></div>

<script>
  function getTheStyle(){
    let elem = document.getElementById("elem-container");
    let theCSSprop = window.getComputedStyle(elem,null).getPropertyValue("height");
    document.getElementById("output").innerHTML = theCSSprop;
   }
  getTheStyle();
</script>
```

`getComputedStyle`返回的对象与从元素的 `style`属性返回的对象具有相同的类型；然而，两个对象具有不同的目的。从`getComputedStyle`返回的对象是只读的，可以用于检查元素的样式（包括由一个`<style>`元素或一个外部样式表设置的那些样式）。`elt.style`对象应用于在特定元素上设置样式。

## 与伪元素一起使用

getComputedStyle 可以从**伪元素**拉取样式信息 (比如，`::after`, `::before`, `::marker`, `::line-marker`）

```` js
<style>
    h3::after {
        content: "rocks!";
    }
</style>

<h3>generated content</h3>

<script>
    let h3 = document.querySelector('h3'),
    result = getComputedStyle(h3, '::after').content;
    alert(`the generated content is: ${result}`);
    console.log(`the generated content is: ${result}`);
    // the generated content is: "rocks!"
</script>
````

