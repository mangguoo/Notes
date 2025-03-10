## 浏览器是如何渲染页面的？

> 当浏览器的网络线程收到HTML文档后，会产生一个渲染任务，并将其传递给渲染主线程的消息队列。
>
> 在事件循环机制的作用下，渲染主线程取出消息队列中的渲染任务，开启渲染流程。

<img src="https://raw.githubusercontent.com/mangguoo/imgRepo/main/img-2/image-20250310235325380.png" alt="image-20250310235325380" style="zoom:25%;" />

### 渲染流程

1. 主线程（Main Thread）

- HTML解析 (Parse)

- 计算样式（Style）

- 布局计算（Layout）

- 分层（Layering）

- 生成绘制指令（Paint Commands）

- 提交给合成线程（Commit）

2. 合成线程（Compositor Thread）

- 分块（Tiling）

- 光栅化（Rasterization）

- 合成（Compositing）

每个阶段都有明确的输入输出，上一个阶段的输出会成为下一个阶段的输入。这样，整个渲染流程就形成了一套组织严密的生产流水线。

### HTML解析 (Parse)

渲染的第一步是**解析HTML**。

解析过程中遇到CSS解析CSS，遇到JS执行JS。为了提高解析效率，浏览器在开始解析前，会启动一个预解析的线程，率先下载HTML中的外部CSS文件和外部的JS文件。

如果主线程解析到`link`位置，此时外部的CSS文件还没有下载解析好，主线程不会等待，继续解析后续的HTML。这是因为下载和解析CSS的工作是在预解析线程中进行的。这就是CSS不会阻塞HTML解析的根本原因。

<img src="https://raw.githubusercontent.com/mangguoo/imgRepo/main/img-2/image-20250311000403129.png" alt="image-20250311000403129" style="zoom:25%;" />

如果主线程解析到`script`位置，会停止解析HTML，转而等待JS文件下载好，并将全局代码解析执行完成后，才能继续解析HTML。这是因为JS代码的执行过程可能会修改当前的DOM树，所以DOM树的生成必须暂停。这就是JS会阻塞HTML解析的根本原因。

<img src="https://raw.githubusercontent.com/mangguoo/imgRepo/main/img-2/image-20250311000548470.png" alt="image-20250311000548470" style="zoom:25%;" />

第一步完成后，会得到DOM树和CSSOM树，浏览器的默认样式、内部样式、外部样式、行内样式均会包含在CSSOM树中。

<img src="https://raw.githubusercontent.com/mangguoo/imgRepo/main/img-2/image-20250311000023967.png" alt="image-20250311000023967" style="zoom: 25%;" />

<img src="https://raw.githubusercontent.com/mangguoo/imgRepo/main/img-2/image-20250311000049436.png" alt="image-20250311000049436" style="zoom: 25%;" />

### 计算样式（Style）

主线程会遍历得到的DOM树，依次为树中的每个节点计算出它最终的样式，称之为Computed Style。

在这一过程中，很多预设值会变成绝对值，比如`red`会变成`rgb(255,0,0)`；相对单位会变成绝对单位，比如`em`会变成`px`。

这一步完成后，会得到一棵带有样式的DOM树。

<img src="https://raw.githubusercontent.com/mangguoo/imgRepo/main/img-2/image-20250311000703800.png" alt="image-20250311000703800" style="zoom:25%;" />

### 布局计算（Layout）

布局阶段会依次遍历DOM树的每一个节点，计算每个节点的几何信息。例如节点的宽高、相对包含块的位置。

大部分时候，DOM树和布局树并非一一对应。原因如下：

1. `display:none`的节点没有几何信息，因此不会生成到布局树；
2. 对于伪元素选择器，虽然他们不存在于DOM树中，但它们拥有几何信息，所以会生成到布局树中。
3. 还有匿名行盒、匿名块盒等等都会导致DOM树和布局树无法一一对应。

- 内容必须在行盒中；
- 行盒和块盒不能相邻;
- 以上两条HTML规则导致了匿名行盒和匿名块盒的产生；

<img src="https://raw.githubusercontent.com/mangguoo/imgRepo/main/img-2/image-20250311000801978.png" alt="image-20250311000801978" style="zoom:25%;" />

### 分层（Layering）

主线程会使用一套复杂的策略对整个布局树中进行分层。

分层的好处在于，将来某一个层改变后，仅会对该层进行后续处理，从而提升效率。

滚动条、堆叠上下文、transform、opacity等样式都会或多或少的影响分层结果，也可以通过`will-change`属性更大程度的影响分层结果。

```css
.container {
    will-change: transform;
} 
/* 加上这个属性，浏览器就会根据自己的策略更大机率将该元素单独分层 */
```

下面是Twitter主页的分层信息：

![image-20250311001414646](https://raw.githubusercontent.com/mangguoo/imgRepo/main/img-2/image-20250311001414646.png)

### 生成绘制指令（Paint Commands）

主线程会**为每个层单独**产生绘制指令集，用于描述这一层的内容该如何画出来。

这一步仅是生成绘制指令集，非常类似于canvas绘制操作，例如：

1. 将画笔移动到10,30位置；
2. 画一个200*300的矩形；
3. 用红色填充矩形
4. ... ...

### 提交给合成线程（Commit）

完成绘制后，主线程将每个图层的绘制信息提交给合成线程，剩余工作将由合成线程完成。

### 分块（Tiling）

合成线程首先对每个图层进行分块，将其划分为更多的小区域。

它会从线程池中拿取多个线程来完成分块工作。

----

分块完成后，进入**光栅化**阶段。

合成线程会将块信息交给 GPU 进程，以极高的速度完成光栅化。

GPU 进程会开启多个线程来完成光栅化，并且优先处理靠近视口区域的块。

光栅化的结果，就是一块一块的位图

---------

最后一个阶段就是**画**了

合成线程拿到每个层、每个块的位图后，生成一个个「指引（quad）」信息。

指引会标识出每个位图应该画到屏幕的哪个位置，以及会考虑到旋转、缩放等变形。

变形发生在合成线程，与渲染主线程无关，这就是`transform`效率高的本质原因。

合成线程会把 quad 提交给 GPU 进程，由 GPU 进程产生系统调用，提交给 GPU 硬件，完成最终的屏幕成像。

## 什么是 reflow？

reflow 的本质就是重新计算 layout 树。

当进行了会影响布局树的操作后，需要重新计算布局树，会引发 layout。

为了避免连续的多次操作导致布局树反复计算，浏览器会合并这些操作，当 JS 代码全部完成后再进行统一计算。所以，改动属性造成的 reflow 是异步完成的。

也同样因为如此，当 JS 获取布局属性时，就可能造成无法获取到最新的布局信息。

浏览器在反复权衡下，最终决定获取属性立即 reflow。

## 什么是 repaint？

repaint 的本质就是重新根据分层信息计算了绘制指令。

当改动了可见样式后，就需要重新计算，会引发 repaint。

由于元素的布局信息也属于可见样式，所以 reflow 一定会引起 repaint。

## 为什么 transform 的效率高？

因为 transform 既不会影响布局也不会影响绘制指令，它影响的只是渲染流程的最后一个「draw」阶段

由于 draw 阶段在合成线程中，所以 transform 的变化几乎不会影响渲染主线程。反之，渲染主线程无论如何忙碌，也不会影响 transform 的变化。
