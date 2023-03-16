# **AbortController** 

> Node.js v15.0.0提供了一个全局实用API AbortController，用于在选定的基于Promise API中发出取消信号。无需引入在所有模块中均可使用，该 API 的实现是基于浏览器中的Web API AbortController

通俗的讲AbortController表示一个控制器对象，允许我们根据需要中止一个或多个Web请求。下面是一个示例，在1秒后会执行ac.abort()方法，将会触发abort事件，并且仅会触发一次，这可通过abortSignal.aborted属性查看前后改变状态：

```js
const ac = new AbortController();

ac.signal.addEventListener('abort', () => {
  console.log('Aborted!');
  console.log('ac.signal.aborted:', ac.signal.aborted);
}, { once: true });

setTimeout(() => ac.abort(), 1000)

console.log('ac.signal.aborted:', ac.signal.aborted);
```

## 中止Promise

> 传递ac.signal中止一个正在运行的Promise，这需要我们为ac.signal注册一个abort事件，做一些处理。之后在任何地方调用ac.abort()中止Promise

使用Promise表示中止操作的任何Web平台APIs都必须遵循以下原则：

- 通过一个signal字典成员接受AbortSignal对象
- 通过reject一个带有"AbortError"DOMException这个类的Promise来表示操作已中止
- 检查AbortSignal对象的aborted标志是否已经被设置，如果是则立即reject，否则：
- 使用中止算法机制来观察对AbortSignal对象的更改，并以不会导致与其他观察者冲突的方式进行观察

```php
function doSomethingAsync({ ac }) {
  return new Promise((resolve, reject) => {
    console.log('task start...');
    if (ac.aborted) {
      return reject(new DOMException('task handler failed', 'AbortError'));
    }
 
    const timer = setTimeout(() => {
      console.log('task end...');
      resolve(1);
    }, 5000);
    ac.signal.addEventListener('abort', () => {
      clearTimeout(timer);
      reject(new DOMException('task handler failed', 'AbortError'));
    }, { once: true });    
  });
}
 
setTimeout(() => ac.abort(), 2000)
try {
  await doSomethingAsync({ ac });
} catch (err) {
  console.error(err.name, err.message); // AbortError task handler failed
}
```