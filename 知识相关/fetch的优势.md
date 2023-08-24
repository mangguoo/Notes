# fetch的优势

> 流是一个很常见的概念，就是允许我们一段一段地接受和处理数据。相对于获取完整的一大块数据再进行后续处理，流只要一段一段地接收并实时处理数据，不需要等所有数据都接收完成再进行处理，缩短了整个操作的耗费时间。流在接收和处理的时候，不需要占用一大块的内存来进行数据的存储，可以在处理后释放掉已经处理好的数据，较少的内存占用是它的特点
>
> 此外流还有管道的概念，我们可以封装一些类似中间件的中间流，用管道将各个流连接起来，在管道的末端就能拿到处理后的数据

<img src="https://cdn.jsdelivr.net/gh/ilmangoi/imgRepo@main/img-2/v2-32191f64dcda7ad21b4f58e91bf84b9e_1440w.png" alt="img" style="zoom:50%;" />

## 数据流管道

这些中间件就是既可以写，又可以读的流，Input进来的是可以读的流，Output出来的是一个可以写的流。这样就引申出来3个名词：

- ReadableStream可读流
- WritableStream可写流
- TransformStream转换流（可读可写流）

这样，就像是一个完整的数据处理流水线。现在Streams API已经在浏览器上逐步实现，能用上流处理的API想必也会越来越多，Fetch API就是最早一批使用Stream API的

## 使用Streams API

Streams API赋予了fetch请求以片段处理数据的能力，让我们可以从网络中一段一段的接收数据并实时处理。而在XMLHttpRequest时代，我们需要等待浏览器把数据完整地接收后，转换为我们需要的格式，然后我们才能进行处理

<img src="https://cdn.jsdelivr.net/gh/ilmangoi/imgRepo@main/img-2/v2-764d70bcec14b6473cf063fd9ef27c9a_1440w.webp" alt="img" style="zoom:50%;" />

现在有了StreamsAPI , 我们一段一段的接收TypedArray的二进制数据，通过对这些底层数据的处理，组合成我们所需要的数据，大大地提高了我们在网络传输到完成处理数据的效率

<img src="https://cdn.jsdelivr.net/gh/ilmangoi/imgRepo@main/img-2/v2-556cc46a85e916ef3ebca8c47cc0d8d8_1440w.webp" alt="img" style="zoom:50%;" />

## 如何产生数据流

```ts
const response: Response = await fetch('https://bar.com/foo');

console.log(response.body); // ReadableStream
```

我们可以通过fetch请求来获取到这个流，在发起fetch请求之后，我们可以得到一个Response对象，这个Response对象中的body，就是一个ReadableStream

## ReadableStream

一个 ReadableStream有下列API:

- `locked`是否锁定到reader
- `cancel()`放弃读取流
- `getReader()`获取一个reader
- `pipeThrough()`进行管道传输
- `pipeTo()`交给最终的可写流
- `tee()`分流

我们要读取一个流的数据，必先有一个reader，所以我们要调用getReader()来获取一个ReadableStreamDefaultReader实例

## ReadableStreamDefaultReader

ReadableStreamDefaultReader实例上提供了如下的方法：

- `closed`
- `cancel()`
- `read()`
- `releaseLock()`

我们使用ReadableStreamDefaultReader来读取流里面的数据，可以调用read()方法，因为每次传输过来的数据时一段一段的，所以我们要多次调用read()方法

```ts
const reader = readableStream.getReader();

(async function (reader: ReadableStreamReader<T>) {
    const { value, done } = await reader.read();

    if (done) {
        return;
    }

    console.log(chunk);
})(reader);
```

其中value参数为这次读取得到的片段，它是一个Uint8Array，通过循环调用reader.read()方法就能一点点地获取流的整个数据

而done参数负责表明这个流是否已经读取完毕，当done为true时表明流已经关闭，不会再有新的数据，此时value的值为undefined

## 取消读取

假如我们在读取流的过程中不想再读取数据，我们可以调用cancel()方法。调用了之后，这个流就不能再被读取

## 释放读取锁

releaseLock()方法用于释放reader对流的锁定

## closed

这个是一个Promise, 它会在流被释放或者是读取完成后resolve/reject，所以我们可以监听到读取结束的状态

```ts
const reader = readableStream.getReader();

reader.closed.then(
    () => {
        console.log('读取完成了');
    },
    () => {
        console.log('读取错误了');
    }
);
```

## 分流

因为一个可读流只能同时有一个reader进行操作，所以我们可以通过调用tee()方法，从一个未锁定的可读流分流为两个可读流。两个可读流是完全独立的。也就是说，这两个可读流的读取与锁定互不相关

<img src="https://picx.zhimg.com/80/v2-46b0bbc248ea3dbd2625f67e919664d0_1440w.webp?source=1940ef5c" alt="img" style="zoom:50%;" />

```ts
const response: Response = await fetch('https://bar.com/foo');

const [streamA, streamB] = response.body.tee();

console.log(streamA); // ReadableStream
console.log(streamB); // ReadableStream
```

我们可以利用其中一个流进行数据读取，另外一个流进行其他操作，例如获取数据的进度

```ts
const response: Response = await fetch('https://bar.com/foo');

const [streamA, streamB] = response.body.tee();

const dataReader = streamA.getReader();

(async function (reader: ReadableStreamReader<T>) {
    const { value, done } = await reader.read();

    if (done) {
        return;
    }

    console.log(chunk);
})(dataReader);

const progressReader = streamB.getReader();

let got = 0;

(async function (reader: ReadableStreamReader<T>) {
    const { value, done } = await reader.read();

    if (done) {
        return;
    }

    const total = response.headers.get('content-length');

    got += value.length;

    console.log(got, total, got / total); // 下载进度
})(progressReader);
```

## 管道方法

pipeTo()和pipeThrough()方法是两个管道方法。可以将当前流指向其他流，最后拿到处理后的数据

pipeTo只能指向WritableStream, pipeThrough只能指向TransformStream

## WritableStream

WritableStream是一个流的终点。它与ReadableStream的原理大致相同，区别是它的数据是写入的，ReadableStream是读取的

## TransformStream

TransformStream是一个可读可写流，它是数据管道中的中间件，属于数据中间层的转化。TransformStream内部会有一个ReadableStream和一个WritableStream。这样的结构让它可以承上启下，上文读取ReadableStream的数据，然后写入到WritableStream中