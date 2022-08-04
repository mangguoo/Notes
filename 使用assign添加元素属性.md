``` js
Object.assign(div.style, {
  width: '50px',
  height: '50px',
  backgroundColor: 'red',
  borderRadius: '50px'
} as CSSStyleDeclaration)
```

- **`Object.assign()`** 方法将所有[可枚举](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/propertyIsEnumerable)（`Object.propertyIsEnumerable()` 返回 true）和[自有](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/hasOwnProperty)（`Object.hasOwnProperty()` 返回 true）属性从一个或多个源对象复制到目标对象，返回修改后的对象。
-  `as`是ts的关键字，用来进行类型断言，断言就相当于告诉编译器这个对象是什么类型，这样vscode就会进行代码提示了。