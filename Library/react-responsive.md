# react-responsive

## 安装

```shell
$ npm install react-responsive --save
```

## 使用示例

### Hooks写法

```jsx
import React from 'react'
import { useMediaQuery } from 'react-responsive'

const Example = () => {
  const isDesktopOrLaptop = useMediaQuery({
    query: '(min-width: 1224px)'
  })
  const isBigScreen = useMediaQuery({ query: '(min-width: 1824px)' })
  const isTabletOrMobile = useMediaQuery({ query: '(max-width: 1224px)' })
  const isPortrait = useMediaQuery({ query: '(orientation: portrait)' })
  const isRetina = useMediaQuery({ query: '(min-resolution: 2dppx)' })

  return <div>
    <h1>Device Test!</h1>
    {isDesktopOrLaptop && <p>You are a desktop or laptop</p>}
    {isBigScreen && <p>You  have a huge screen</p>}
    {isTabletOrMobile && <p>You are a tablet or mobile phone</p>}
    <p>Your are in {isPortrait ? 'portrait' : 'landscape'} orientation</p>
    {isRetina && <p>You are retina</p>}
  </div>
}
```

### Components写法

```jsx
import MediaQuery from 'react-responsive'

const Example = () => (
  <div>
    <h1>Device Test!</h1>
    <MediaQuery minWidth={1224}>
      <p>You are a desktop or laptop</p>
      <MediaQuery minWidth={1824}>
        <p>You also have a huge screen</p>
      </MediaQuery>
    </MediaQuery>
    <MediaQuery minResolution="2dppx">
      {/* 还可以将函数作用child */}
      {(matches) =>
        matches
          ? <p>You are retina</p>
          : <p>You are not retina</p>
      }
    </MediaQuery>
  </div>
)
```

## useMediaQuery的属性

> useMediaQuery的第一个属性就是媒体查询对象，为了符合react的命名规范，媒体查询名称统一使用小驼峰命名方式
>
> 当我们写任何数值没有给单位时，useMediaQuery会自动给我们加上单位，这个单位默认为px

```js
const isDesktopOrLaptop = useMediaQuery({ minWidth: 1224 })
const isBigScreen = useMediaQuery({ minWidth: 1824 })
const isTabletOrMobile = useMediaQuery({ maxWidth: 1224 })
const isPortrait = useMediaQuery({ orientation: 'portrait' })
const isRetina = useMediaQuery({ minResolution: '2dppx' })
```

## 指定设备

> 在测试时我们需要观察组件在不同设备上的显示情况，而不仅仅只是通过自动检测。这在无法检测到这些设置(SSR)或用于测试的Node环境中尤其有用
>
> useMediaQuery的第二个参数就是用来指定设备的，这个属性会始终被应用，即使可以通过第一个参数检测到的情况下

**Keys:**

`orientation`, `scan`, `aspectRatio`, `deviceAspectRatio`, `height`, `deviceHeight`, `width`, `deviceWidth`, `color`, `colorIndex`, `monochrome`, `resolution` and `type`

**Types:**

`type`可以是其中之一: `all`, `grid`, `aural`, `braille`, `handheld`, `print`, `projection`, `screen`, `tty`, `tv` or `embossed`

**例子:**

```jsx
import { useMediaQuery } from 'react-responsive'

const Example = () => {
  const isDesktopOrLaptop = useMediaQuery(
     { minDeviceWidth: 1224 },
     { deviceWidth: 1600 } // `device` prop
  )

  return (
    <div>
      {isDesktopOrLaptop &&
        <p>isDesktopOrLaptop将始终为true，即使设备长度小于1224px，这是因为我们用“deviceWidth: 1600”覆盖了设备设置</p>
      }
    </div>
  )
}
```

SSR: 我们还可以通过React Context将设备传递给组件树中的每个useMediaQuery hook。这将简化在Node环境中的服务器端渲染和测试

Context只是一个常规的react上下文组件，它接受一个“value”支持传递给consuming组件

```jsx
import { Context as ResponsiveContext } from 'react-responsive'
import { renderToString } from 'react-dom/server'
import App from './App'

...
  const mobileApp = renderToString(
    <ResponsiveContext.Provider value={{ width: 500 }}>
      <App />
    </ResponsiveContext.Provider>
  )
...
```

## `onChange`事件

> useMediaQuery还有第三个参数，它是一个change事件的回调，该处理程序将在媒体查询的值更改时调用

### Hooks写法

```jsx
import React from 'react'
import { useMediaQuery } from 'react-responsive'

const Example = () => {

  const handleMediaQueryChange = (matches) => {
    // 根据媒体查询的值，matches 将为 true 或 false
  }
  const isDesktopOrLaptop = useMediaQuery(
    { minWidth: 1224 }, undefined,  handleMediaQueryChange
  );

  return (
    <div></div>
  )
}
```

### Components写法

```jsx
import React from 'react'
import MediaQuery from 'react-responsive'

const Example = () => {

  const handleMediaQueryChange = (matches) => {
    // 根据媒体查询的值，matches 将为 true 或 false
  }

  return (
    <MediaQuery minWidth={1224} onChange={handleMediaQueryChange}>
      ...
    </MediaQuery>
  )
}
```