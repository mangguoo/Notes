# languageDetector

> 这是一个`i18next`语言检测插件，用于检测浏览器中的用户语言，支持如下方式：
>
> - cookie (set cookie i18next=LANGUAGE)
> - sessionStorage (set key i18nextLng=LANGUAGE)
> - localStorage (set key i18nextLng=LANGUAGE)
> - navigator (set browser language)
> - querystring (append `?lng=LANGUAGE` to URL)
> - htmlTag (add html language tag <html lang="LANGUAGE" ...)
> - path (http://my.site.com/LANGUAGE/...)
> - subdomain ([http://LANGUAGE.site.com/](http://language.site.com/)...)

## 安装

```shell
$ npm install i18next-browser-languagedetector
```

## 使用

```js
import i18next from 'i18next';
import LanguageDetector from 'i18next-browser-languagedetector';

i18next.use(LanguageDetector).init(i18nextOptions);
```

## 探测器选项

### 默认选项

```json
{
  // 定义检测顺序与应该在哪些地方检测
  order: ['querystring', 'cookie', 'localStorage', 'sessionStorage', 'navigator', 'htmlTag', 'path', 'subdomain'],

  // 语言探测器所探测的键
  lookupQuerystring: 'lng',
  lookupCookie: 'i18next',
  lookupLocalStorage: 'i18nextLng',
  lookupSessionStorage: 'i18nextLng',
  lookupFromPathIndex: 0,
  lookupFromSubdomainIndex: 0,

  // 把用户语言缓存在什么地方
  caches: ['localStorage', 'cookie'],
  excludeCacheFor: ['cimode'], // 定义不持久化的语言 (cookie, localStorage)

  // 设置cookie的 expire 和 domain
  cookieMinutes: 10,
  cookieDomain: 'myDomain',

  // 定义带lang属性htmlTag为哪个元素
  htmlTag: document.documentElement,

  // cookie选项
  cookieOptions: { path: '/', sameSite: 'strict' }
}
```

### 自定义选项

```js
import i18next from 'i18next';
import LanguageDetector from 'i18next-browser-languagedetector';

i18next.use(LanguageDetector).init({
  detection: options,
});
```

```js
import LanguageDetector from 'i18next-browser-languagedetector';
const languageDetector = new LanguageDetector(null, options);
```

```js
import LanguageDetector from 'i18next-browser-languagedetector';
const languageDetector = new LanguageDetector();
languageDetector.init(options);
```