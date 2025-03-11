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

- 最终呈现到屏幕

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

它会从线程池中拿取多个线程来完成分块工作。具体流程如下：
1. 计算每个 Tile 覆盖哪些图层（Layer）

    例如，一个256×256的Tile可能同时包含背景图层、文本图层和前景图层。

    浏览器会遍历所有图层，判断哪些图层影响该Tile。

2. 计算该Tile在这些图层中的相对位置

    每个图层都有自己的局部坐标系，Tile需要确定它在不同图层中的偏移量。

    例如，一个Tile可能位于某个图层的(100, 50)坐标处，但在另一个图层的(200, 0)。

3. 记录影响该Tile的绘制指令

    并不会直接拆分绘制指令，而是筛选出与该Tile相关的绘制指令，稍后进行光栅化。

4. 等待GPU进行光栅化

    只有当某个Tile需要显示（如滚动进入视口），才会实际执行绘制。

#### 分块的优势

1. 只光栅化（Rasterization）需要显示的Tiles

    优化点：避免不必要的计算

    浏览器不会一次性光栅化所有Tiles，而是只处理当前视口可见的Tiles。

    对于即将进入视口的Tiles，可能会提前渲染（Pre-rasterization），保证滚动时流畅。

    不可见区域的Tiles（如页面底部或远离视口的部分）不会立即处理，节省GPU资源。

📌 示例：

假设屏幕显示3x4个Tiles（共 12 个）。如果页面有100个Tiles，GPU只渲染当前屏幕上的12个，其余88个不处理。

---

2. Tiles作为GPU纹理存储，避免重复渲染

    优化点：减少CPU → GPU数据传输，提高重绘效率

    光栅化后的Tiles会作为纹理（Texture）存储在GPU显存中，下次重绘时可以直接复用。

    这样可以避免CPU重新计算所有绘制指令，提高帧率。

📌 示例：

如果页面背景是复杂的渐变，首次渲染时GPU计算并存储Tile纹理，后续滚动时无需重新计算，只需移动现有Tiles。

---

3. 只更新变化的Tiles，减少重绘范围（Partial Repaint）

    优化点：局部重绘，而不是整个屏幕重绘

    当页面局部发生变化（如输入框闪烁、按钮hover），GPU只会重绘受影响的Tiles，而不是整个页面。

    通过Dirty Region检测，浏览器可以判断哪些Tiles发生了变化，仅更新这些部分。

📌 示例：

一个网页的按钮发生hover效果，GPU只更新这个按钮所在的Tile，页面其他部分不会重新渲染。

---

4. Tiles进行异步合成（Async Compositing）

    优化点：滚动和动画更流畅，不受主线程影响

    合成线程（Compositor Thread）可以独立控制Tile的显示位置，无需等待主线程响应。

    例如，页面滚动时，GPU 只是移动已有的 Tiles，而不是重新渲染它们。

    这样即使主线程繁忙（如执行 JavaScript），页面滚动仍然保持流畅。

📌 示例：

当你快速滚动页面时，GPU只是调整已渲染Tiles的位置，而不会重新计算页面布局或样式。

---

5. GPU加速特效（如硬件合成、3D 变换）

    优化点：使用GPU专长的操作，提高渲染性能

    一些CSS特性（如transform, opacity, will-change）可以让Tiles进入GPU加速模式，避免CPU计算负担。

    浏览器会为这些元素创建独立的合成层（Compositing Layer），并让GPU直接控制它们的位置和透明度。

📌 示例：

translate3d(0, 100px, 0)只会让GPU 动Tile纹理，而不会触发页面重排（Reflow）。

### 光栅化（Rasterization）

合成线程会将块信息交给GPU进程，以极高的速度完成光栅化。GPU进程会开启多个线程来完成光栅化，并且优先处理靠近视口区域的块。光栅化的结果，就是一块一块的位图。

但是要注意，光栅化是针对每个Layer（图层）的，不是整个页面。GPU并不会直接生成合成后的最终位图，而是为每个独立的合成层生成位图（Texture）。

因为不同的图层可能会发生透明、变换（transform）、遮挡等操作，需要在合成阶段进行处理。

### 合成（Compositing）

合成（Compositing）」是浏览器渲染的最后阶段，这一步GPU不会重新计算像素，而是按照正确的层级顺序，将不同的「合成层」合成到最终画面上。同时进行透明混合（Alpha Blending）、遮挡裁剪、变换（如 transform、opacity）等操作。合成阶段可以复用已有的纹理，仅通过变换和混合实现高效的动画效果。

当浏览器检测到某个元素需要独立合成（如transform, opacity, will-change），它会创建一个合成层（Compositing Layer），该层的像素内容由GPU纹理存储。这样，GPU可以直接操作纹理，而不需要重新计算像素。

#### 合成层的优势

如果被操作变幻的元素已经启用了GPU合成层（Compositing Layer）（如transform、opacity、will-change），则GPU走了一条更快的路径：

1. 合成线程（Compositor Thread）将直接触发GPU合成

    - 不触发光栅化（Rasterization）
    - 不生成新的Tiles
    - 不重新计算像素

2. GPU直接变换已有的纹理（Texture）：

    - 该元素已经被光栅化为纹理（Texture），存储在显存中
    - GPU只对纹理应用矩阵变换（Matrix Transformation），而不是重新计算像素
    - 这一步是超快的硬件操作，比重新光栅化快很多

3. GPU合成（Compositing）并输出到屏幕

    - 直接渲染最终画面，不需要重新计算像素
    - 所有层级信息由合成树（Compositing Tree）管理，保证正确顺序
    - 处理完成后，整个帧渲染完成


#### 「合成」流程

> 合成流程主要由合成线程（Compositor Thread和GPU线程（GPU Process）协同完成。大致步骤如下：

1. 计算合成顺序

    合成线程（Compositor Thread）会：

    - 遍历「合成树（Compositing Tree）」，确定每个合成层的绘制顺序
    - 确定层级关系（z-index, stacking context）
    - 计算哪些合成层需要更新（比如transform变化的元素）
    - 剔除不可见区域，避免浪费GPU资源

    📌 示例

    ```html
    <div class="bg"></div>
    <div class="box" style="transform: translateX(100px);"></div>
    ```

    浏览器构建 「合成树」：

    ```shell
    合成树：
    Root (body)
    ├── bg (普通合成层)
    ├── box (独立合成层，启用 GPU)
    ```

    合成线程会分析：

    - bg是底层，要先绘制
    - box是transform变换的元素，单独作为合成层
    - box需要变换，但不需要重新光栅化

2. 发送合成指令给GPU

    合成线程不会自己绘制像素，而是发送「合成指令（Compositing Commands）」给GPU线程。GPU线程根据这些指令，调整每个合成层的位置、缩放、旋转等

    ```js
    drawLayer(bg, position=(0,0))
    drawLayer(box, transform=translateX(100px))
    ```

    此时，GPU 并不会重新计算像素，而是仅对box进行变换操作！

3. GPU处理 & 合成

    - 将所有合成层转换为纹理（Texture）
    - 应用变换（Transform）
    - 按照合成树顺序，正确绘制这些纹理
    - 执行混合（Blending）、遮挡计算（Overlapping）、抗锯齿（Anti-aliasing）等
    - 最终输出到屏幕（Frame Buffer）

    📌 示例: 假设box有opacity: 0.5，GPU会：

    - 先渲染bg
    - 叠加box（使用透明度混合）
    - 最终输出完整的画面

4. 发送合成结果到显示设备

    - GPU把最终帧数据发送到Frame Buffer
    - 操作系统（如 Windows Compositor, macOS Quartz）再把Frame Buffer传输给显示器
    - 屏幕刷新（通常是60Hz或120Hz）后，用户就能看到最终画面

## 什么是 reflow？

reflow的本质就是重新计算layout树。当进行了会影响布局树的操作后，需要重新计算布局树，会引发reflow。

为了避免连续的多次操作导致布局树反复计算，浏览器会合并这些操作，当JS代码全部完成后再进行统一计算。**所以改动属性造成的reflow是异步完成的。**

也同样因为如此，当JS获取布局属性时，就可能造成无法获取到最新的布局信息。因此浏览器在反复权衡下，最终决定获取属性立即reflow。

## 什么是repaint？

repaint的本质就是重新根据分层信息计算了绘制指令。当改动了可见样式后，就需要重新计算，会引发repaint。

由于元素的布局信息也属于可见样式，所以reflow一定会引起repaint。
