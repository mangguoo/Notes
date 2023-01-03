## HTML

### Html常见标签

#### meta

```html
<meta charset="utf-8">
<-- 指定字符集 -->
    
<meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scaleble=no">
<-- 显示窗口自适应 -->
```

#### a[href, target]

- _blank: 在新窗口中打开被链接文档
- _self: 默认。在相同的框架中打开被链接文档
- _parent: 在父框架集中打开被链接文档
- _top: 在整个窗口中打开被链接文档
- framename: 在指定的框架中打开被链接文档

#### img[src, alt]

- alt：替换内容(即图片无法显示时要显示的内容)
- src： 图片路径

#### table

**table标签:** 

```html
<table width="300">
<caption>表格一</caption>
<colgroup>
  <col span="1" style="background-color: red;">
  <col span="2" style="background-color: blue;">
</colgroup>
<thead>
  <tr>
      <td>a</td>
      <td>a</td>
      <td>a</td>
  </tr>
</thead>
<tfoot>
  <tr>
      <td>a</td>
      <td>a</td>
      <td>a</td>
  </tr>
</tfoot>
<tbody>
  <tr>
      <td>a</td>
      <td>a</td>
      <td>a</td>
  </tr>
</tbody>
</table>
```

**表格属性:**

- table标签属性

	- border： 设置表格边框，不需要写单位，默认单位为px；
- cellspacing： 设置单元格之间的距离，不需要写单位，默认单位为px；
- cellpadding： 设置单元格的内边距，不需要写单位，默认单位为px；

- \<table>, \<tr>, \<td>通用属性


  - width： 设置表格的宽度；
  - height： 设置表格的高度；
  - align： 用在table标签上，是设置table表格整体居中的，而用在tr或者td上是设置里边的文本内容居中；`align=“left/center/right”`

- \<tr>, \<td>通用属性

	- valign： 设置单元格中文字垂直对齐方式；
  +  valign=“top/middle/bottom”

- td标签属性

	- colspan：合并列；
- rowspan：合并行；

- \<colgroup>, \<col>标签属性: colgroup标签用于对表格中的列进行组合，以便对其进行格式化，而col标签表示表格中的其中一列或其中几列

	- span： 规定列组应该横跨的列数；

注意: 上面的属性，除border-collapse外，其它的全是table标签专有属性。而专有属性是必须写在标签上的，而不能写进style中。

```html
<table border="1" 
    width="300px" 
    align="center" 
    cellspacing="10" 
    cellpadding="10"
    style="border-collapse: separate;">
    <tr align="center" valign="middle" height="100px">
        <td colspan="3">a</td>
    </tr>
    <tr align="left" valign="top" height="100px">
        <td rowspan="2">a</td>
        <td>a</td>
        <td>a</td>
    </tr>
    <tr align="right" valign="bottom" height="100px">
        <td>a</td>
        <td>a</td>
    </tr>
</table>
```

#### form

**form:**

- action：处理表单提交的 URL。
- enctype：当 method 属性值为 post 时，enctype 就是将表单的内容提交给服务器的 MIME 类型 。可能的取值有：
  - application/x-www-form-urlencoded：未指定属性时的默认值。
  - multipart/form-data：当表单包含 type=file 的 <input> 元素时使用此值。
  - text/plain：出现于 HTML5，用于调试。
    =>  method：浏览器使用这种 HTTP 方式来提交 表单。可能的值有：
  - post：指的是 HTTP POST 方法；表单数据会包含在表单体内然后发送给服务器。
  - get：指的是 HTTP GET 方法；表单数据会附加在 action 属性的 URL 中，并以 '?' 作为分隔符。

- novalidate：此布尔值属性表示提交表单时不需要验证表单。

- target：表示在提交表单之后，在哪里显示响应信息。在 HTML 4 中，这是一个 frame 的名字/关键字对：
  - \_self：默认值。在相同浏览上下文中加载。
  - \_blank：在新的未命名的浏览上下文中加载。
  - \_parent：在当前上下文的父级浏览上下文中加载，如果没有父级，则与 _self 表现一致。
  - \_top：在最顶级的浏览上下文中（即当前上下文的一个没有父级的祖先浏览上下文），如果没有父级，则与 _self表现一致。

> 注意： 这几个值可被 <button>、<input type="submit"> 或 <input type="image"> 元素上的 formaction、formenctype、formmethod、formnovalidate、formtarget 属性覆盖。

**input：**

- name：input 的名称，与表单数据一起提交。
- value：input 的初始值。它定义的值与表单数据的提交按钮相关联。当表单中的数据被提交时，这个值便以参数的形式被递送至服务器。
- type：该属性是input标签里唯一的必须输入的属性，当然，也可以不填，默认为type = “text”。
- required：标记一个字段是否为必须。如果一个字段被标记为required = “required”（严格模式下），或者required（宽松模式下）(ps：下面属性的写法同理，就不一一写出了)，并且这个字段的值为空，或者填入的值是无效值，那么这个表单不能提交。什么是无效值？看pattern属性。
- readonly：该属性会让表单空控件不可编辑。这里的不可编辑不是禁用，只是不能编辑文本而已。
- disabled：该属性会禁用一个表单元素。
- maxlength：该属性用于限制用户输入的最大字数限制。
- min max：这两个属性用于日期date时间time等输入，还有number和range。顾名思义，它们的作用是限制最大值，最小值。
- pattern：该属性包含了一个JavaScript风格的正则表达式，输入的内容必须完全匹配该正则表达式，不然就算输入的内容无效。
- step：步长的作用是提供一个可以上下点的按钮，比如当前数字是1，你设置了step = “5”，点一下上的按钮，就变成6了。
- placeholder：该属性一般是用来提示用户输入的，当用户真的输入了文字之后，会被输入的文字覆盖。
- size：控制input长度，但是长度可以直接使用CSS控制。
- autocomplete：这个属性表示这个控件的值是否可被浏览器自动填充。
  - off: 用户必须手动填值，或者该页面提供了自己的自动补全方法。浏览器不对此字段自动填充。
  - on: 浏览器可以根据用户先前的填表情况对此字段自动填值。

- autofocus：该属性指的是表示这个表单控件在页面载入的时候自动获得焦点。
- form：指定跟自身相关联的表单。值必须为本文档内的表单的 ID，如果未指定，就是跟当前所在的表单元素相关联。代码如下：

```html
<form id="form1"></form>
<p>
    <label for="admin">admin</label>
    <input type="text" id="admin" form="form1"/>
</p>
```

![image-20221222232601561](https://cdn.jsdelivr.net/gh/ilmangoi/imgRepo@main/img/image-20221222232601561.png)

**label:**

- for：即和 <label> 元素在同一文档中的可关联标签的元素的 id。 
  - 文档中第一个 id 值与 <label> 元素 for 属性值相同的元素，如果可关联标签（labelable），则为已关联标签的控件，其标签就是这个 <label> 元素。
  - 如果这个元素不可关联标签，则 for 属性没有效果。
  - 如果文档中还有其他元素的 id 值也和 for 属性相同，for 属性对这些元素也没有影响。

- form：指定跟自身相关联的表单。值必须为本文档内的表单的 ID，如果未指定，就是跟当前所在的表单元素相关联。

**button:**

- name：button 的名称，与表单数据一起提交。
- value：button 的初始值。它定义的值与表单数据的提交按钮相关联。当表单中的数据被提交时，这个值便以参数的形式被递送至服务器。
- form：指定跟自身相关联的表单。值必须为本文档内的表单的 ID，如果未指定，就是跟当前所在的表单元素相关联。
- disabled：此布尔属性表示用户不能与 button 交互。
- autofocus：一个布尔属性，用于指定当页面加载时按钮有输入焦点，只有一个表单关联元素可以指定该属性。
- type：button 的类型。可选值：
  - submit:  此按钮将表单数据提交给服务器。
  - reset:  此按钮重置所有组件为初始值。
  - button: 此按钮没有默认行为。

**textarea:**

- name：元素的名称。
- form：指定跟自身相关联的表单。值必须为本文档内的表单的 ID，如果未指定，就是跟当前所在的表单元素相关联。
- required：提示用户这个元素的内容必填。
- disabled：禁用文本域。默认为 false。
- readonly：不允许用户修改元素内文本。和 disabled 属性不同的是，这个能让用户点击和选择元素内的文本。如果在表单里，这个元素的值还是会跟随表单一起提交。
- rows：元素的输入文本的行数（显示的高度）。
- cols：文本域的可视宽度。必须为正数，默认为 20。
- maxlength：允许用户输入的最大字符长度 (Unicode) ，未指定表示无限长度。
- minlength：允许用户输入的最小字符长度 (Unicode) 。
- placeholder：向用户提示可以在控件中输入的内容。 在渲染提示时，占位符文本中的回车符(\r) 或换行符 (\n) 一定会被作为行断（换行）处理。
- autocomplete：是否使用浏览器的记忆功能自动填充文本。可能的值有：
  
    - off: 不使用浏览器的记忆自动填充，使用者必须输入他们想要输入的所有内容。或者网页提供了自己的自动填充方法。
    
     +  on: 浏览器根据用户之前输入的内容或者习惯，在用户输入的时候给出相应输入提示。

- autofocus：页面加载完毕之后是否自动给本元素添加焦点。

**fieldset>legend:**

- name：元素分组的名称。
- disabled：如果设置了这个属性，<fieldset> 的所有子代表单控件也会继承这个属性。注意，<legend> 中的表单元素不会被禁用。
- form：指定跟自身相关联的表单。值必须为本文档内的表单的 ID，如果未指定，就是跟当前所在的表单元素相关联。

**select>optgroup>option:**

- select

	- name：该属性规定了控件的名称。
	- form：指定跟自身相关联的表单。值必须为本文档内的表单的 ID，如果未指定，就是跟当前所在的表单元素相关联。
	- disabled：这个布尔值的属性表示用户不能与该表单控件交互。
	- required：一个布尔值属性，表示必须选中一个有非空字符串值的选项。
	- autofocus：这个布尔值属性能够让一个对象在页面加载的时候获得焦点。一个文档中只有一个对象可以有这个属性。
	- multiple：这个布尔值属性表示列表中的选项是否支持多选。没有声明该值时，一次只能选中一个选项。声明这个属性后，大多数浏览器都会显示一个可滚动的列表框，而非一个下拉菜单。
	- size：如果控件显示为滚动列表框（如声明了 multiple），则此属性表示为控件中同时可见的行数。

- option

	- value：这个属性的值表示该选项被选中时提交给表单的值。如果省略了这个属性，值就从选项元素的文本内容中获取。
	- disabled：如果设置了这个布尔属性，则该选项不可选。
	- label：这个属性是用于表示选项含义的文本。如果 label 属性没有定义，它的值就是元素文本内容。
	- selected：这个布尔属性存在时表明这个选项是否一开始就被选中。

- optgroup

  - disabled：如果设置了这个布尔值，则不能选择这个选项组中的任何选项。
  - label：选项组的名字，浏览器用以在用户界面中标记选项。使用这个元素时必须加上这个属性。

**禁用表单:**

> 表单元素被禁用意味着它不可编辑，也不会随着 <form> 一起提交。并且也不会接收到任何浏览器事件，如鼠标点击或与聚焦相关的事件。默认情况下，浏览器会将这样的表单控件展示为灰色。

**案例:**

```html
<form target="_balnk" method="get">
    <fieldset>
        <legend>用户登录</legend>
        <label for="username">账号：</label>
        <input type="text" id="username" name="username"><br>
        <label for="password">密码：</label>
        <input type="password" id="password" name="password"><br>
    </fieldset>

    <label for="male">男</label>
    <input type="radio" id="male" name="sex" value="male"><br>
    <label for="famale">女</label>
    <input type="radio" id="famale" name="sex" value="famale"><br>

    <label for="dino-select">Choose a dinosaur:</label>
    <select id="dino-select">
        <optgroup label="Theropods">
            <option>Tyrannosaurus</option>
            <option>Velociraptor</option>
            <option>Deinonychus</option>
        </optgroup>
        <optgroup label="Sauropods">
            <option>Diplodocus</option>
            <option>Saltasaurus</option>
            <option>Apatosaurus</option>
        </optgroup>
    </select><br>

    <button type="submit">提交</button>
    <button type="reset">清空</button>
</form>
```

#### HTML5新增标签

- section：章节标签；
- article：文章标签；
- nav：导航标签；
- aside：侧边栏标签，表示一些不太重要的内容；
- header/footer：头尾标签；
- em/strong：强调标签，这两个标签都有默认样式，em是斜体，strong是粗体；
- i：i标签在html5中不再做为斜体使用，而是做为icon图标；
  `<i class="iconfont icon-ashbin"></i>`

### Html元素分类

#### 块级元素 block

- 如div，他会独占一行，并且可以设置宽高

#### 行内元素 inline

- 如span，他可以和其他元素共同存在于一行当中，没有固定的形状，因此他不能设置宽高

#### inline-block

- 如select，他不会独占一行，但是他可以向block元素一样可以设置宽高

### Html元素嵌套

1. 块级元素可以包含行内元素；
2. 块级元素不一定能包含块级元素，例如p>p是不合法的；
3. 行内元素一般不能包含块级元素，例如a>div是合法的；

