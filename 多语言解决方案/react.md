- **App.tsx**

```tsx
import React, { Component, Suspense } from 'react';
import { useTranslation, withTranslation, Trans } from 'react-i18next';
import logo from './logo.svg';
import './App.css';
import './i18n';

// 类组件使用hoc
class LegacyWelcomeClass extends Component {
  render() {
    const { t } = this.props;
    return <h2>{t('title')}</h2>;
  }
}
const Welcome = withTranslation()(LegacyWelcomeClass);

// 使用Trans组件
function MyComponent() {
  return (
    <Trans i18nKey="description.part1">
      To get started, edit <code>src/App.js</code> and save to reload.
    </Trans>
  );
}

// 函数组件使用hooks
function Page() {
  const { t, i18n } = useTranslation();

  const changeLanguage = (lng) => {
    i18n.changeLanguage(lng);
  };

  return (
    <div className="App">
      <div className="App-header">
        <img src={logo} className="App-logo" alt="logo" />
        <Welcome />
        <button type="button" onClick={() => changeLanguage('de')}>de</button>
        <button type="button" onClick={() => changeLanguage('en')}>en</button>
      </div>
      <div className="App-intro">
        <MyComponent />
      </div>
      <div>{t('description.part2')}</div>
    </div>
  );
}

const Loader = () => (
  <div className="App">
    <img src={logo} className="App-logo" alt="logo" />
    <div>loading...</div>
  </div>
);

// 这里使用suspense组件来捕捉Page组件中翻译还没有加载的情况
export default function App() {
  return (
    <Suspense fallback={<Loader />}>
      <Page />
    </Suspense>
  );
}
```

- **src/i18n.js**

```ts
import i18next from 'i18next';
import { initReactI18next } from 'react-i18next';
import Backend from 'i18next-http-backend';
import LanguageDetector from 'i18next-browser-languagedetector';
// import ns1 from './en/ns1.json';
// import ns2 from './en/ns2.json';

export const defaultNS = 'ns1';

// export const resources = {
//   en: {
//     ns1,
//     ns2,
//   }
// };

i18next
.use(Backend)
.use(LanguageDetector)
.use(initReactI18next)
.init({
  // lng: 'en', // 由于使用的是detector，所以这些不需要写
  // resources,
  debug: true,
  fallbackLng: 'en',
  defaultNS,
  interpolation: {
    escapeValue: false,
  },
});

export default i18next;
```

- **public/locales/en/ns1.json**

```json
{
  "title": "Welcome to react using react-i18next fully type-safe",
  "description": {
    "part1": "This is a simple example.",
    "part2": "😉"
  }
}
```

- **public/locales/en/ns2.json**

```json
{
  "description": {
    "part1": "In order to infer the appropriate type for t function, you should use type augmentation to override the Resources type.",
    "part2": "Check out the @types/i18next to see an example."
  }
}
```

- **public/locales/de/ns1.json**

```json
{
  "title": "Willkommen zu react und react-i18next",
  "description": {
    "part1": "Um loszulegen, ändere <1>src/App(DE).js</1> und speichere um neuzuladen.",
    "part2": "Wechsle die Sprache zwischen deutsch und englisch mit Hilfe der beiden Schalter."
  }
}
```

- **public/locales/de/ns2.json**

```json
{
  "description": {
    "part1": "Um loszulegen.",
    "part2": "Wechsle die Sprache zwischen deutsch und englisch mit Hilfe der beiden Schalter."
  }
}
```

- **component/Comp1.tsx**

```tsx
import { useTranslation } from "react-i18next";

function Comp1() {
  const {t} = useTranslation();

  return (
    <div className="App">
      <p>{t('title')}</p>
      <p>{t('description.part1')}</p>
      <p>{t('description.part2')}</p>
    </div>
  );
}

export default Comp1;
```

- **component/Comp2.tsx**

```tsx
import { useTranslation } from "react-i18next";

function Comp2() {
  const {t} = useTranslation('ns2');

  return (
    <div className="App">
      <p>{t('description.part1')}</p>
      <p>{t('description.part2')}</p>
    </div>
  );
}

export default Comp2;
```

- **component/Comp3.tsx**

```tsx
import { useTranslation } from "react-i18next";

function Comp3() {
  const {t} = useTranslation(['ns1', 'ns2']);

  return (
    <div className="App">
      <p>{t('ns1:description.part1')}</p>
      <p>{t('ns2:description.part2')}</p>
    </div>
  );
}

export default Comp3;
```

