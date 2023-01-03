# classnames

> 在react中如果想要让一个元素上存在多个className，可以使用模板字符串，也可以使用js脚本动态计算，但是这些写法都很麻烦，这里我们就可以使用classnames这个库，它可以有效的减少我们添加多个classNames时的操作：

```jsx
import classnames from 'classnames'

<div
  className={classNames(
    props.className, // 字符串，或者数组
    { // 对象，value为true则表示启用
      [styles.buttonBig]: size === 'big',
      [styles.buttonMiddle]: size === 'middle',
      [styles.buttonSmall]: size === 'small',
    },
  )}
>
  hello world!
</div>
```

