# qs

## qs.parse

> 允许在查询字符串中创建嵌套对象，方法是用方括号[]包含子键的名称

```js
qs.parse('foo[bar]=baz')
    
{
    foo: { bar: 'baz' }
}
```

> 当使用 plainObjects 选项时，返回的对象是通过 Object.create (null)创建，因此原型方法不会存在于它上面

```js
qs.parse('a[hasOwnProperty]=b', { plainObjects: true })

{ 
    a: { hasOwnProperty: 'b' } 
}
```

> 如果将 allowPrototype 设置为 true，将忽略会覆盖对象原型上的属性的参数

```js
qs.parse('a[hasOwnProperty]=b', { allowPrototypes: true })

{ 
    a: { hasOwnProperty: 'b' } 
}
```

> URI 编码的字符串也可以工作

```js
qs.parse('a%5Bb%5D=c')

{
    a: { b: 'c' }
}
```

> 对象是可以嵌套的，默认情况下，嵌套对象时，qs 只会深度解析多达5个子对象。这意味着，如果尝试解析像`a[b][c][d][e][f][g][h][i] = j`这样的字符串，结果对象将是

```js
qs.parse('a[b][c][d][e][f][g][h][i]=j')

{
    a: {
        b: {
            c: {
                d: {
                    e: {
                        f: {
                            '[g][h][i]': 'j'
                        }
                    }
                }
            }
        }
    }
};
```

> 可以通过向 qs.parse (string，[ options ])传递深度选项来覆盖这个深度

```js
qs.parse('a[b][c][d][e][f][g][h][i]=j', { depth: 1 })

{ 
    a: { 
        b: { 
            '[c][d][e][f][g][h][i]': 'j' 
        } 
    } 
}
```

> 默认情况下 qs 只解析最多1000个参数。这可以通过传递参数限制选项来覆盖

```js
qs.parse('a=b&c=d', { parameterLimit: 1 })
{ a: 'b' }
```

> 若要绕过前导问号，请使用 ignoreQueryPrefix

```js
qs.parse('?a=b&c=d', { ignoreQueryPrefix: true });
{ a: 'b', c: 'd' }
```

> 可以传递一个可选的分隔符

```js
qs.parse('a=b;c=d', { delimiter: ';' });
{ a: 'b', c: 'd' }
```

> 分隔符也可以是正则表达式

```js
qs.parse('a=b;c=d,e=f', { delimiter: /[;,]/ });
{ a: 'b', c: 'd', e: 'f' }
```

> 选项 allowDots 可用于启用点符号

```js
qs.parse('a.b=c', { allowDots: true });
{ a: { b: 'c' } }
```

> 还可以使用下面的表示法解析数组:

```js
qs.parse('a[]=b&a[]=c');
{ a: ['b', 'c'] }
```

```js
qs.parse('a[1]=c&a[0]=b');
{ a: ['b', 'c'] }
```

> 请注意，数组中的索引和对象中的键之间的唯一区别是，方括号之间的值必须是创建数组的数字。当创建具有特定索引的数组时，qs 会将稀疏数组压缩到只保留现有值

```js
qs.parse('a[1]=b&a[15]=c');
{ a: ['b', 'c'] }
```

> 可以使用 allowSparse 选项来解析稀疏数组

```js
qs.parse('a[1]=2&a[3]=5', { allowSparse: true });
{ a: [, '2', , '5'] }
```

> 请注意，空字符串也是一个值，将被保留

```js
qs.parse('a[]=&a[]=b');
{ a: ['', 'b'] })

qs.parse('a[0]=b&a[1]=&a[2]=c');
{ a: ['b', '', 'c'] })
```

> qs还将数组中指定索引的最大索引限制为20。索引大于20的任何数组成员将转换为以索引为键的对象，可以通过传递 arrayLimit 选项覆盖此限制。若要完全禁用数组解析，请将 parseArarray 设置为 false

```js
qs.parse('a[100]=b');
{ a: { '100': 'b' } }
```

```js
qs.parse('a[1]=b', { arrayLimit: 0 });
{ a: { '1': 'b' } }
```

```js
qs.parse('a[]=b', { parseArrays: false });
{ a: { '0': 'b' } }
```

> 如果你混合了符号，qs 会把两个项合并成一个对象

```js
qs.parse('a[0]=b&a[b]=c');
{ a: { '0': 'b', b: 'c' } }
```

> 还可以创建对象数组

```js
qs.parse('a[][b]=c');
{ a: [{ b: 'c' }] }
```

> 有些人使用逗号创建数组，qs 也可以解析它(这不能转换嵌套对象，例如 a = { b: 1} ，{ c: d })

```js
qs.parse('a=b,c', { comma: true })
{ a: ['b', 'c'] }
```

> 默认情况下，所有值都被解析为字符串。这种行为不会改变

```js
qs.parse('a=15&b=true&c=null');
{ a: '15', b: 'true', c: 'null' }
```

## qs.stringify

> 在字符串化时，默认情况下，qs 对输出进行 URI 编码。可以通过将 encode 选项设置为 false 来禁用此编码。还可以通过将 encodeValuesOnly 选项设置为 true 来禁用键的编码:

```js
qs.stringify({ a: 'b' })
'a=b'
qs.stringify({ a: { b: 'c' } })
'a%5Bb%5D=c'
```

```js
qs.stringify({ a: { b: 'c' } }, { encode: false })
'a[b]=c'
```

```js
qs.stringify(
    { a: 'b', c: ['d', 'e=f'], f: [['g'], ['h']] },
    { encodeValuesOnly: true }
);
'a=b&c[0]=d&c[1]=e%3Df&f[0][0]=g&f[1][0]=h'
```

> qs还提供了自定义的编码/解码器：

```js
var encoded = qs.stringify({ a: { b: 'c' } }, { encoder: function (str, defaultEncoder, charset, type) {
    if (type === 'key') {
        return // Encoded key
    } else if (type === 'value') {
        return // Encoded value
    }
}})
```

```js
var decoded = qs.parse('x=z', { decoder: function (str, defaultDecoder, charset, type) {
    if (type === 'key') {
        return // Decoded key
    } else if (type === 'value') {
        return // Decoded value
    }
}})
```

> 当数组被字符串化时，默认情况下会给它们显式的索引。可以通过将 index 选项设置为 false 来覆盖此选项，还可以使用 arrayFormat 选项来指定输出数组的格式
>
> **注意:** 在使用 arrayFormat 设置为“comma”时，还可以将 commaRoundTrip 选项设置为 true 或 false，以便在单项数组上附加[] ，这样就避免编码后回不去不情况

```js
qs.stringify({ a: ['b', 'c', 'd'] });
// 'a[0]=b&a[1]=c&a[2]=d'
```

```js
qs.stringify({ a: ['b', 'c', 'd'] }, { indices: false });
// 'a=b&a=c&a=d'
```

```js
qs.stringify({ a: ['b', 'c'] }, { arrayFormat: 'indices' })
// 'a[0]=b&a[1]=c'
qs.stringify({ a: ['b', 'c'] }, { arrayFormat: 'brackets' })
// 'a[]=b&a[]=c'
qs.stringify({ a: ['b', 'c'] }, { arrayFormat: 'repeat' })
// 'a=b&a=c'
qs.stringify({ a: ['b', 'c'] }, { arrayFormat: 'comma' })
// 'a=b,c'
```

> 当对象被字符串化时，默认情况下它们使用括号表示法，通过将 allowDots 选项设置为 true，可以覆盖此选项来使用点表示法

```js
qs.stringify({ a: { b: { c: 'd', e: 'f' } } });
// 'a[b][c]=d&a[b][e]=f'
```

```js
qs.stringify({ a: { b: { c: 'd', e: 'f' } } }, { allowDots: true });
// 'a.b.c=d&a.b.e=f'
```

> 空字符串和空值将省略该值，但等号(=)仍然保留

```js
qs.stringify({ a: '' })
'a='
```

> 没有值的键(如空对象或数组)将不返回任何值

```js
equal(qs.stringify({ a: [] }), '');
equal(qs.stringify({ a: {} }), '');
equal(qs.stringify({ a: [{}] }), '');
equal(qs.stringify({ a: { b: []} }), '');
equal(qs.stringify({ a: { b: {}} }), '');
```

> 设置为未定义的属性将完全省略

```js
qs.stringify({ a: null, b: undefined })
'a='
```

> 可以选择在查询字符串前面加上问号

```js
qs.stringify({ a: 'b', c: 'd' }, { addQueryPrefix: true })
'?a=b&c=d'
```

> 分隔符也可以用 stringify 重写

```js
qs.stringify({ a: 'b', c: 'd' }, { delimiter: ';' })
'a=b;c=d'
```

> 如果只想重写 Date 对象的序列化，可以提供 seralizeDate 选项

```js
var date = new Date(7);
qs.stringify({ a: date })
'a=1970-01-01T00:00:00.007Z'.replace(/:/g, '%3A')

qs.stringify({ a: date }, { serializeDate: function (d) { return d.getTime(); } })
'a=7'
```

> 可以使用排序选项来影响参数键的顺序

```js
function alphabeticalSort(a, b) {
    return a.localeCompare(b);
}
qs.stringify({ a: 'c', z: 'y', b : 'f' }, { sort: alphabeticalSort })
'a=c&b=f&z=y'
```

> 可以使用 filter 选项来限制字符串化输出中将包含哪些键

```js
function filterFunc(prefix, value) {
    if (prefix == 'b') {
        // 返回undefined会忽略掉一个属性
        return;
    }
    if (prefix == 'e[f]') {
        return value.getTime();
    }
    if (prefix == 'e[g][0]') {
        return value * 2;
    }
    return value;
}
qs.stringify({ a: 'b', c: 'd', e: { f: new Date(123), g: [2] } }, { filter: filterFunc });
// 'a=b&c=d&e[f]=123&e[g][0]=4'
qs.stringify({ a: 'b', c: 'd', e: 'f' }, { filter: ['a', 'e'] });
// 'a=b&e=f'
qs.stringify({ a: ['b', 'c', 'd'], e: 'f' }, { filter: ['a', 0, 2] });
// 'a[0]=b&a[2]=d'
```

> 还可以使用过滤器为用户定义的类型注入自定义序列化。假设正在使用一些 api，它们希望查询字符串的格式为范围

```js
class Range {
    constructor(from, to) {
        this.from = from;
        this.to = to;
    }
}
qs.stringify(
    {
        range: new Range(30, 70),
    },
    {
        filter: (prefix, value) => {
            if (value instanceof Range) {
                return `${value.from}...${value.to}`;
            }
            // serialize the usual way
            return value;
        },
    }
);
// range=30...70
```

## 处理null

> 默认情况下，空值被视为空字符串

```js
qs.stringify({ a: null, b: '' })
'a=&b='
```

> 解析不区分带等号和不带等号的参数。两者都转换为空字符串

```js
qs.parse('a&b=')
{ a: '', b: '' }
```

> 若要区分 null 值和空字符串，请使用 strictNullProcessing 标志。在结果字符串中，空值没有 = 符号

```js
qs.stringify({ a: null, b: '' }, { strictNullHandling: true })
'a&b='
```

> 要解析的值不使用 = back，而要返回 null，可以使用 strictNullProcessing 标志

```js
qs.parse('a&b=', { strictNullHandling: true })
{ a: null, b: '' }
```

> 若要完全跳过具有 null 值的呈现键，请使用 skipNull 标志:

```js
qs.stringify({ a: 'b', c: null}, { skipNulls: true })
'a=b'
```