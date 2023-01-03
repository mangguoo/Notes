# sharp

## 安装

```bash
npm install sharp
# 或者
yarn add sharp
```

## 创建图像

sharp 创建图像常用的有三种方法，**直接生成、从文件读取、Buffer**

### **直接生成**

给 sharp 传入一个包含 create 对象的对象，create 里的内容就是新建图像的配置

```js
const sharp = require('sharp');

const image = sharp({
    create: {
        // 宽度
        width: 300,
        // 高度
        height: 200,
        // 通道数
        channels: 4,
        // 指定背景色
        background: 'black' // 或者 '#000000' 或者更详细的 rgba 配置对象
    }
});
```

### **从文件读取**

提供一个路径，sharp 会去这个路径找图片，找到了就读进来：

```js
const image = sharp('./origin.png');
```

### **从 Buffer 读取**

传入一个 buffer，sharp 会用这个 buffer 进行实例化，这个方法是最最常用的创建图像的方法，原因下文会提到

```js
// sharp 可以解析 svg，很实用
const buffer = Buffer.from(
    '<svg xmlns="http://www.w3.org/2000/svg" width="150" height="150"></svg>'
)
const image = sharp(buffer)
```

这种方式也适合拿来读网络请求来的图片：

```js
// 用 axios 请求网络图片
const axios = require('axios');

axios.get('http://xxxxx/xx/origin.png', {
    responseType: 'arraybuffer'
}).then(resp => {
    const iamge = sharp(resp.data)
})
复制代码
```

## 保存图像

保存图片有两种方法，一种是直接保存成图片：

```js
const buffer = Buffer.from(
    '<svg xmlns="http://www.w3.org/2000/svg" width="150" height="150"></svg>'
)

sharp(buffer)
    .png()
    .toFile('./result.png')
    .then(() => console.log('保存完成'))
```

第二种是转换成 buffer，然后怎么处理就随你了。如果是Node程序toFile()方法会直接把将参数作为图片名称将图像文件保存在程序的工作目录中。而在浏览器环境中是不可以的，必须转换成Buffer对象，然后再进行后续的下载操作:

```js
sharp('./origin.png')
    .png()
    .toBuffer()
    .then(buffer => {
        console.log('buffer', buffer)
    })
```

这里有两点需要注意：

- **`toBuffer` 和 `toFile` 都是异步方法**
- **如果实例化的图片是无格式的，要先调方法设置其文件类型**

比如如果保存的时候弹出来了这个错误：

```js
Error: Unsupported output format ./result.png
```

这说明要保存的图片是没有设置格式的（此时的格式为原始格式 raw），需要使用 `.png`、`.jpeg` 之类的方法设置格式

## 修改图像

sharp 支持的图像修改操作包括Resize和一大堆其他处理方法。看名字就知道是搞啥的了。注意这里的方法都是 **同步方法** 并且支持链式调用，所以可以这样写：

```js
const sharp = require('sharp');

const buffer = Buffer.from(`
    <svg>
        <rect width="100" height="100" fill="slategrey"/>
        <circle cx="60" cy="106" r="50" fill="yellow"/>
    </svg>
`)

sharp(buffer)
    .flip()
    .rotate(45)
    .resize(500)
    .toFile('./result.png')
    .then(() => console.log('保存完成'));
```

### 调整图片大小

> 可以使用resize()方法调整图片大小，而不从中剪切任何内容的过程，这会影响图像文件大小

```js
const sharp = require("sharp");

async function resizeImage() {
  try {
    await sharp("sammy.png")
      .resize({
        width: 150,
        height: 97
      })
      .toFile("sammy-resized.png");
  } catch (error) {
    console.log(error);
  }
}

resizeImage();
```

### 更改图像格式和压缩图像

> 使用`toFormat()`方法可以转换图片的格式，并且可以对图片进行体积压缩。图像压缩是在不降低质量的情况下减小图像文件大小的过程
>
> `toFormat()`方法的第一个参数是一个字符串，其中包含要将图像转换为的图像格式。第二个参数是一个可选对象，包含增强和压缩图像的输出选项
>
> 要压缩图像，需要传递一个包含布尔值的`mozjpeg`属性。当将其设置为 true 时，sharp 使用 mozjpeg 默认值来压缩图像而不牺牲质量
>
> 注意：关于 toFormat() 方法的第二个参数，每种图像格式都采用具有不同属性的对象。 例如，仅在 JPEG 图像上接受 mozjpeg 属性。但是，其他图像格式具有等效选项，例如`quality`、`compression`和`lossless`。 请务必参阅[官方文档](https://links.jianshu.com/go?to=https%3A%2F%2Fsharp.pixelplumbing.com%2Fapi-output)以了解您正在压缩的图像格式可接受的选项类型

```js
const sharp = require("sharp");

async function resizeImage() {
  try {
    await sharp("sammy.png")
      .resize({
        width: 150,
        height: 97
      })
      .toFormat("jpeg", { mozjpeg: true })
      .toFile("sammy-resized-compressed.jpeg");
  } catch (error) {
    console.log(error);
  }
}

resizeImage();
```

### 裁剪图像

> 可以使用`extract()`方法来裁剪图像,`extract()`方法接收一个对象，该对象中可以包含如下参数：
>
> - width：要裁剪的区域的宽度。
> - height：要裁剪的区域的高度。
> - top：要裁剪的区域的垂直位置。
> - left：要裁剪的区域的水平位置。

```js
const sharp = require("sharp");

async function cropImage() {
  try {
    await sharp("sammy.png")
      .extract({ width: 500, height: 330, left: 120, top: 70  })
      .toFile("sammy-cropped.png");
  } catch (error) {
    console.log(error);
  }
}

cropImage();
```

### 设置灰度

```js
const sharp = require("sharp");

async function grayImage() {
  try {
    await sharp("sammy.png")
      .grayscale()
      .toFile("sammy-grayscale.png");
  } catch (error) {
    console.log(error);
  }
}

grayImage();
```

### 旋转图像

> 默认情况下，sharp 将旋转图像的背景设为黑色。要移除黑色背景，您需要传递一个对象作为第二个参数以使背景透明
>
> 该对象具有一个background属性，该属性包含一个定义 RGBA 颜色模型的对象。 RGBA 代表红色、绿色、蓝色和 alpha
> 要使 alpha 属性起作用，必须确保设置 r、g、 b 的值。将 r、g 、b 值设置为 0 会创建黑色。要创建透明背景，必须先定义颜色，然后可以将 alpha 设置为 0 使其透明

```js
const sharp = require("sharp");

async function rotateImage() {
  try {
    await sharp("sammy.png")
      .rotate(33, { background: { r: 0, g: 0, b: 0, alpha: 0 } })
      .toFile("sammy-rotated.png");
  } catch (error) {
    console.log(error);
  }
}

rotateImage();
```

### 高斯模糊

> 该方法接受一个 0.3-1000之间的参数

```js
const sharp = require("sharp");

async function blurImage() {
  try {
    await sharp("sammy.png")
      .blur(4)
      .toFile("sammy-blurred.png");
  } catch (error) {
    console.log(error);
  }
}

blurImage();
```

### 添加文字

> sharp没有向图像添加文本的原生方式。要添加文本，首先编写代码以使用可缩放矢量图形 (SVG) 绘制文本。创建 SVG 图像后，使用composite()方法对它们进行合并

```js
const sharp = require("sharp");

async function addTextOnImage() {
  try {
    const width = 750;
    const height = 483;
    const text = "Sammy the Shark";

    const svgImage = `
    <svg width="${width}" height="${height}">
      <style>
      .title { fill: #001; font-size: 70px; font-weight: bold;}
      </style>
      <text x="50%" y="50%" text-anchor="middle" class="title">${text}</text>
    </svg>
    `;
    const svgBuffer = Buffer.from(svgImage);
    const image = await sharp("sammy.png")
      .composite([
        {
          input: svgBuffer,
          top: 0,
          left: 0,
        },
      ])
      .toFile("sammy-text-overlay.png");
  } catch (error) {
    console.log(error);
  }
}

addTextOnImage();
```



## 拼接图像

拼接使用的是 `composite` 方法。这个方法接收一个数组，数组每个元素都代表了一个要拼接到源图上的图片，其中的 `input` 属性代表图片本体，`blend` 代表拼接方法，`left` 代表拼接到的位置距离底图左边的（像素）宽度，`top` 代表拼接到的位置距离底图顶部的（像素）宽度：

```js
const sharp = require('sharp');

const run = async function () {
    // 100 * 300 的黑色长方形底图
    const background = await sharp({ create: {
        height: 100, width: 300, channels: 4, background: 'black'
    }}).png();

    // 100 * 100 的蓝色瓦片
    const blueTile = await sharp({ create: {
        height: 100, width: 100, channels: 4, background: 'blue'
    }}).png().toBuffer();

    background
        .composite([{ input: blueTile, blend: 'atop', left: 0, top: 0 }])
        .toFile('./testResult.png');
}

run();
```

然后就会生成：

![img](https://cdn.jsdelivr.net/gh/ilmangoi/imgRepo@main/img/9779a0b382ec47d78a8e9a1b57ee1985~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)

这里的 `blend：atop` 应该是最常用的，就是单纯的把一个图叠到另一张图上。其他支持的属性见 [这里](https://link.juejin.cn?target=https%3A%2F%2Fsharp.pixelplumbing.com%2Fapi-composite%23composite)，详细介绍可以看 [这里](https://link.juejin.cn?target=https%3A%2F%2Fwww.cairographics.org%2Foperators%2F)

------

这里需要注意的是：**`composite` 方法接收的 `input` 属性不能是一个 sharp 实例**！例如上面的例子里，我先用 sharp 创建了要拼接的图片，但是需要在最后使用 `toBuffer` 方法将其转换为 buffer 然后才能传入 composite 进行拼接，下面是文档中的介绍：

![img](https://cdn.jsdelivr.net/gh/ilmangoi/imgRepo@main/img/dfe611dc95a74fffa3b280a2f27664a5~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)

可以看到，**要么传入 buffer 或者路径字符串，要么就是用 create 创建新的图片**，唯独不能使用 sharp 实例

------

除此之外 `composite` 还有个比较大的问题：**不支持链式调用！**如果我们链式一下，尝试用三块蓝色铺满背景图：

```js
background
    .composite([{ input: blueTile, blend: 'atop', left: 0, top: 0 }])
    .composite([{ input: blueTile, blend: 'atop', left: 100, top: 0 }]) // <= 指定了往右偏移
    .composite([{ input: blueTile, blend: 'atop', left: 200, top: 0 }])
    .toFile('./testResult.png');
复制代码
```

在期望里，它的最终结果应该是全部被蓝色填满了。但是最后生成的却是这样的：

![img](https://cdn.jsdelivr.net/gh/ilmangoi/imgRepo@main/img/10b50b9da5244f7287263b98b8508738~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)

**只有最后一个 composite 生效了**，要想解决这个问题有两种办法：

第一个办法是 **不使用链式调用，把所有的图片都放在同一个 composite 接收的数组里**：

```js
background
    .composite([
        { input: blueTile, blend: 'atop', left: 0, top: 0 },
        { input: blueTile, blend: 'atop', left: 100, top: 0 },
        { input: blueTile, blend: 'atop', left: 200, top: 0 }
    ])
    .toFile('./testResult.png');
```

这样最大的问题是需要同时把要拼接的所有图片载入到内存里，当要拼接亿像素级别的图像时，对于小内存机器可能不太友好。并且 **composite 是同步操作**，一定要小心不要让他卡死了进程

第二个办法是 **每次 composite 之后 toBuffer 然后重新实例化**。如果需要处理一张超大图片，最好还是用第一种方法，先少量并发把每行拼接好保存到本地。等所有行都拼接好之后再把保存路径同时传给大底图把所有行拼在一起

------

不仅是 composite 链式调用有问题，**composite 和其他操作链式调用也有问题**，例如下面的例子，先拼接、缩放完之后再左右颠倒一下，期望里蓝色方块也会被同步缩放，但是最后结果却和先缩放再拼接一样：

```js
background
    .composite([{ input: blueTile, blend: 'atop', left: 0, top: 0 }])
    .resize(500)
    .flop(true) // <= 左右颠倒
    .toFile('./testResult.png');
复制代码
```

然后就会发现颠倒的操作没生效：

![img](https://cdn.jsdelivr.net/gh/ilmangoi/imgRepo@main/img/b638c203de874a65b62837c115d0329b~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)

在这种情况下就比较推荐第二种解决方法了。如下，每次操作之后先转换成 buffer，然后再做下一个操作：

```js
let buffer = await background
    .composite([{ input: blueTile, blend: 'atop', left: 0, top: 0 }])
    .toBuffer();

buffer = await sharp(buffer)
    .composite([{ input: blueTile, blend: 'atop', left: 0, top: 0 }])
    .resize(500)
    .toBuffer();

sharp(buffer)
    .flop(true)
    .toFile('./testResult.png');
复制代码
```

最后的结果就和我们预期的一致了：

![img](https://cdn.jsdelivr.net/gh/ilmangoi/imgRepo@main/img/f6bc037a75ae48d0abfe19b4e14381fc~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)

## 修改透明度

经常会有需求是调整一个图片的整体透明度，但是 sharp 这里对透明度操作的支持不太好，在 api 文档的通道操作里我们可以看到：

![image.png](https://cdn.jsdelivr.net/gh/ilmangoi/imgRepo@main/img/67d4d98032794a52995e4f47f860625d~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)

`remvoeAlpha` 是直接砍掉透明通道，`ensureAlpha` 是如果透明通道不存在的话就补上。**没有直接调整透明度的 api**，这里把解决方法贴上，来自官方 issue 区：

```js
/**
 * 给图片添加透明度
 * 由于 sharp.ensureAlpha 只会为没有透明通道的图片添加透明通道
 * 当一个图片已经有透明通道时是无法使用该方法调整透明度的
 * 这里用的方法来自 @see https://github.com/lovell/sharp/issues/618#issuecomment-532293211
 *
 * @param sharp 要添加的图片
 * @param opacity 透明度 0 - 255，255 是完全不透明
 * @returns 添加透明度之后的图片
 */
export const addOpacity = function (sharp: Sharp, opacity = 128): Sharp {
    return sharp.composite([{
        input: Buffer.from([255, 255, 255, opacity]),
        raw: { width: 1, height: 1, channels: 4 },
        tile: true,
        blend: 'dest-in'
    }]);
};
```

## 修改像素上限

在创建 sharp 实例或者 composite 的时候如果遇到了下面的问题：

```arduino
Error: Input image exceeds pixel limit
```

那么说明你要读取的图像过大超过了默认的像素上限，你可以通过指定 `limitInputPixels` 的方式调整像素上限，如下：

```js
// 新建时
sharp('./bigImage.png', { limitInputPixels: 1000000000 }).toFile('result.png');

// 拼接时
sharp('./origin.png')
    .composite([{ input: 'bigImage.png', limitInputPixels: 1000000000 }])
    .toFile('result.png')
复制代码
```

## 读取图像元数据

> 图像元数据是嵌入到图像中的文本，其中包括有关图像的信息，例如其类型、宽度和高度

```js
const sharp = require("sharp");

async function getMetadata() {
  try {
    const metadata = await sharp("sammy.png").metadata();
    console.log(metadata);
  } catch (error) {
    console.log(`An error occurred during processing: ${error}`);
  }
}
```

输出结果示例：

```json
{
  format: 'png',
  width: 750,
  height: 483,
  space: 'srgb',
  channels: 3,
  depth: 'uchar',
  density: 72,
  isProgressive: false,
  hasProfile: false,
  hasAlpha: false
}
```

