# next-i18next

## 安装依赖

- i18next
- next-i18next
- react-i18next
- next
- react
- crc
- i18next-scanner

## 配置文件

### next.config.js

```js
// @ts-check
const { i18n } = require('./next-i18next.config');

const nextConfig = {
  reactStrictMode: true,
  swcMinify: true,
  i18n, // 添加i18n配置
}

module.exports = nextConfig
```

### next-i18next.config.js

```js
const locales = ['en', 'zh-CN', 'zh-HK', 'ko', 'fr', 'es', 'pt', 'ja'];

module.exports = {
  locales,
  i18n: {
    locales: locales, // 支持的语言
    defaultLocale: 'en', // 默认选用的语言
    defaultNS: 'translation', // 默认的命名空间
    localePath: './locales', // 语言文件所在的位置
    localeDetection: false,
  },
};
```

### i18next-scanner.config.js

```js
const fs = require('fs')
const chalk = require('chalk')
const { crc32 } = require('crc')
const { locales } = require('./next-i18next.config')

module.exports = {
  input: [ // 要扫描的路径
    'pages/**/*.{js,jsx,ts,tsx}',
    'components/**/*.{js,jsx,ts,tsx}',
    '!app/**/*.spec.{js,jsx}',
    '!app/i18n/**',
    '!**/node_modules/**',
  ],
  output: './', // 输出路径
  options: {
    debug: false,
    func: false,
    trans: false,
    lngs: locales, // 要生成的语言集合
    defaultLng: 'en', // 默认语言
    ns: ['translation'], // 要生成的命名空间集合
    defaultNs: 'translation', // 默认的命名空间
    resource: {
      loadPath: 'locales/{{lng}}/{{ns}}.json', // 加载目录
      savePath: 'locales/{{lng}}/{{ns}}.json', // 保存目录
      jsonIndent: 2,
      lineEnding: '\n',
    },
    nsSeparator: false, // namespace separator
    keySeparator: false, // key separator
    removeUnusedKeys: true,
    interpolation: {
      prefix: '{{',
      suffix: '}}',
    },
  },
  transform: function customTransform(file, enc, done) { // 转换函数
    'use strict'
    const parser = this.parser
    const content = fs.readFileSync(file.path, enc)
    let count = 0
    parser.parseFuncFromString(content, { list: ['t'] }, (key, options) => { // 根据t函数进行解析
      options.defaultValue = key // 设置options的默认值，options中还有可能包括命名空间，上层key等内容
      let haskKey = `K${crc32(key).toString(16)}` // 把t函数中包含的英文内容进行crc校验
      parser.set(haskKey, options) // 以校验内容为key，英文为value进行保存
      ++count
    })
    if (count > 0) {
      console.log(`i18next-scanner: count=${chalk.cyan(count)}, file=${chalk.yellow(JSON.stringify(file.relative))}`)
    }
    done()
  },
}
```

### 配置扫描指令

```json
"scripts": {
  "dev": "next dev",
  "build": "next build",
  "start": "next start",
  "lint": "next lint",
  "scan": "i18next-scanner --config i18next-scanner.config.js",
  "stop": "pm2 stop nuxt-waas",
  "pm2": "pm2 start npm --name nuxt-waas -- start"
},
```

## 使用i18n进行国际化

### pages/_app.tsx

```tsx
import type { AppProps } from 'next/app'
import { appWithTranslation } from 'next-i18next'
// import nextI18NextConfig from '../next-i18next.config.js'

const MyApp = ({ Component, pageProps }: AppProps) => (
  <Component {...pageProps} />
)

// 入口组件必须使用appWithTranslation进行包裹
export default appWithTranslation(MyApp /*, nextI18NextConfig */)
```

### pages/_document.tsx

```tsx
import Document, {
  Html,
  Head,
  Main,
  NextScript,
} from 'next/document'
import type { DocumentProps } from 'next/document'
import i18nextConfig from '../next-i18next.config'

type Props = DocumentProps & {
  // add custom document props
}

class MyDocument extends Document<Props> {
  render() {
    const currentLocale =
      this.props.__NEXT_DATA__.locale ??
      i18nextConfig.i18n.defaultLocale
    return (
      /* 在模板中设置语言 */
      <Html lang={currentLocale}> 
        <Head>
          <meta charSet="utf-8" />
        </Head>
        <body>
          <Main />
          <NextScript />
        </body>
      </Html>
    )
  }
}

export default MyDocument
```

## 切换语言

> 在next应该中使用了next-i18next后，每个路由地址都将存在于每个地区中，并且next框架知道它们是相同的(也就是说`en/about`和`zh-cn/about`会定位到同一个页面)
>
> 如果想要导航到一个特定的语言环境，我们必须提供一个`locale`属性给`Link`组件，否则，它将根据浏览器的`Accept-Language`请求头(用来说明访问者希望采用的语言或语言组合，这样的话用户就可以根据自己偏好的语言来定制不同的内容)来进行设置

### hooks/useLocale.ts

```ts
import {useTranslation} from "next-i18next";
import {crc32} from "crc";
import {TOptions} from "i18next";

// 自定义useLocale进行暴露t函数，这样可以自定义t函数的行为，我们这里是让h函数进行crc编码生成key之后，再返回来查询value
const useLocale = (NS: string | string[]) => {
  const i18n = useTranslation(NS); // useTranslation可以接收一个参数，表示它返回的t函数要从哪个命名空间中去取value

  const t = (key:string, options?: TOptions) => {
    const hashKey = `K${crc32(key).toString(16)}`
    const value = i18n.t(hashKey, options);
    // 没有找到相应key的时候，t函数会直接返回hashKey
    if(value === hashKey) {
      return key;
    }
    return value;
  }

  return {t};
}

export default useLocale
```

### pages/index.tsx

```tsx
import Link from 'next/link'
import { useRouter } from 'next/router'
import type { GetStaticProps, InferGetStaticPropsType } from 'next'

import useLocale from "../hooks/useLocale";

import { useTranslation, Trans } from 'next-i18next'
import { serverSideTranslations } from 'next-i18next/serverSideTranslations'

type Props = {
  // Add custom props here
}

const Homepage = (
  _props: InferGetStaticPropsType<typeof getStaticProps>
) => {
  const router = useRouter()
  const { t } = useLocale('common') // 使用自定义hook

  // eslint-disable-next-line @typescript-eslint/no-unused-vars
  const onToggleLanguageClick = (newLocale: string) => {
    const { pathname, asPath, query } = router
    router.push({ pathname, query }, asPath, { locale: newLocale })
  }

  const changeTo = router.locale === 'en' ? 'de' : 'en'

  return (
    <>
      <main>
        <div style={{ display: 'inline-flex', width: '90%' }}>
          {t("hello world!")}
        </div>
        <hr style={{ marginTop: 20, width: '90%' }} />
        <div>
          <Link href="/" locale={changeTo}>
            <button>{t('change-locale', { changeTo })}</button>
          </Link>
          {/* alternative language change without using Link component
          <button onClick={() => onToggleLanguageClick(changeTo)}>
            {t('change-locale', { changeTo })}
          </button>
          */}
        </div>
      </main>
    </>
  )
}

// or getServerSideProps: GetServerSideProps<Props> = async ({ locale })
// 这个函数的作用是给当前页面注册相应的命名空间，比如英语的translation命名空间
export const getStaticProps: GetStaticProps<Props> = async ({ 
  locale,
}) => ({
  props: {
    ...(await serverSideTranslations(
      locale ?? 'en', 
      ['translation'],
      null, 
      ['en', 'no']
    )),
  },
})

export default Homepage
```

默认情况下，next-i18next会在每次初始请求时将所有name space发送到客户机。对于内容较少的小型应用程序来说是一种合适的方法，但是如果多语言字段很多，就可以根据路由来拆分命名空间，这样就可以将每个页面所需的名称空间数组传递到serverSideTransations，这样在用户访问时，就只会收到指定命名空间的字段

默认情况下，next-i18next会在请求中向客户端发送当前语言环境的字段。这有助于减少发送到客户端的初始有效负载的大小。然而在某些情况下，在运行时也可能需要其他语言的翻译。例如，当使用useTransationhook的getFixedT时

```js
// fix language to german
const de = i18next.getFixedT('de');
de('myKey');

// or fix the namespace to anotherNamespace
const anotherNamespace = i18next.getFixedT(null, 'anotherNamespace');
anotherNamespace('anotherNamespaceKey'); // no need to prefix ns i18n.t('anotherNamespace:anotherNamespaceKey');
```

这里就需要加载额外的语言设置，只需将所需语言设置为数组作为最后一个参数传递给serverSideTransations即可

## 自定义生成翻译文件

> 这一步完成之后就可以扫描页面了`npm run scan`，它会自动帮我们生成local语言文件，但是这个时候翻译文件中只有英文，我们需要手动把这些英文翻译成对应的语言，然后就可以在切换语言时看到各种语言了

```txt
├─en
│   translation.json
├─es
│   translation.json
├─fr
│   translation.json
├─ja
│   translation.json
├─ko
│   translation.json
├─pt
│   translation.json
├─zh-CN
│   translation.json
└─zh-HK
    translation.json
```



