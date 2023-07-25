# satisfies

> ts代码包括自动推导出的类型、手动声明的类型两种

```typescript
type Obj = {
	a: number
	b: number | string
	[key: string]: any
}
```

key:string那部分是索引签名，也就是任意的key为string，value为任意类型的索引都可以加

```ts
const obj: Obj = {
  a: 1,
  b: '1', 
  c: 1
}
obj.b = 1
obj.d = 1
```

这样手动声明类型之后，它只会提示声明的索引，动态添加的字段是没有代码提示的，但是如果让ts自动推导类型，是有类型提示的	

```ts
const obj = {
  a: 1,
  b: '1', 
  c: 1
}
obj.b = 1 // 报错
obj.d = 1 // 报错
```

虽然自动推导有很多优势，但是他没有类型约束，定义的时候写的什么类型，他就会被推导为什么类型。可以发现自动推导和手动声明各有各的优势和劣势：

- 手动声明类型可以让类型更严格，但是遇到动态类型没有代码提示
- 自动推导的类型更加灵活，但是没有类型规范，比较容易出错

而ts的新版本中加入了一个新语法satisfies：

```ts
const obj = {
  a: 1,
  b: '1', 
  c: 1
} satisfies Obj
obj.b = 1 // 报错
obj.d = 1 // 报错
```

使用satisfies指定类型，它会自动推导变量的类型，而不是声明的类型，增加灵活性，同时还可以对这个推导出的类型做类型检查，保证安全

但是satisfies的方式也有它的问题，由于他使用的是推导出来的类型，这就会导致它不能动态扩展索引，同时还会对联合类型进行类型缩小