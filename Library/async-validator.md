# async-validator

## 安装

```bash
npm i async-validator
```

## 基础使用

```js
import Schema from 'async-validator';

const descriptor = {
  name: {
    type: 'string',
    required: true,
    validator: (rule, value) => value === 'muji',
  },
  age: {
    type: 'number',
    asyncValidator: (rule, value) => {
      return new Promise((resolve, reject) => {
        if (value < 18) {
          reject('too young');  
        } else {
          resolve();
        }
      });
    },
  },
};

const validator = new Schema(descriptor);

validator.validate({ name: 'muji' }, (errors, fields) => {
  if (errors) {
   // 验证失败，错误是所有错误的数组
   // Fields 是由字段名称为键的对象，每个字段对应一个错误数组
    return handleErrors(errors, fields);
  }
  // 验证通过
});

// Promise写法
validator.validate({ name: 'muji', age: 16 }).then(() => {
  // 验证通过
}).catch(({ errors, fields }) => {
  return handleErrors(errors, fields);
});
```

## API

### Validate

```js
function(source, [options], callback): Promise
```

- `source`: 要验证的对象(必需)
- `options`: 描述验证处理选项的对象(可选)
- `callback`: 验证完成时调用的回调函数(可选)

该方法将返回一个Promise对象:

- `then()`，验证通过
- `catch({ errors, fields })`，验证失败：errors是所有错误的数组，fields是由字段名为键的对象，每个字段对应一个错误数组

### Options

- `suppressWarning`: 布尔值，是否禁止关于无效值的内部警告
- `first`: 布尔值，当第一个验证规则发生错误时就调用回调，不再处理任何验证规则。如果验证涉及多个异步调用(例如，数据库查询)，并且只需要第一个错误，请使用此选项
- `FirstFields`: Boolean | String []，当指定字段的第一个验证规则发生错误时调用回调，不再处理同一字段的其他验证规则

### rules

> rules是执行验证的函数:
>
> ```js
> function(rule, value, callback, source, options)
> ```

- `rule`: 源descriptor中与正在验证的字段名相对应的验证规则

- `value`: 正在验证的源对象属性的值

- `callback`: 验证完成后调用的回调函数。它期望接收一个 Error 实例数组来指示验证失败

  如果检查是同步的，则可以直接返回 false 或 Error 或 Error Array，而不需要调用callback进行返回

- `source`: 传递给验证方法的源对象

- `options`: 其他选项

```js
import Schema from 'async-validator';
const descriptor = {
  name(rule, value, callback, source, options) {
    const errors = [];
    if (!/^[a-z0-9]+$/.test(value)) {
      errors.push(new Error(
        util.format('%s must be lowercase alphanumeric characters', rule.field),
      ));
    }
    // 返回一个数组，因为validator函数的callback，接收一个fileld参数，这个参数包含每个字段对象的错误，这个错误就是保存在一个数组中
    return errors;
  },
};
const validator = new Schema(descriptor);
validator.validate({ name: 'Firstname' }, (errors, fields) => {
  if (errors) {
    return handleErrors(errors, fields);
  }
  // 验证通过
});
```

针对单个字段使用多个验证规则进行验证通常很有用，而只需要把validator参数写成一个数组即可，例如:

```js
const descriptor = {
  email: [
    { type: 'string', required: true, pattern: Schema.pattern.email },
    { 
      validator(rule, value, callback, source, options) {
        const errors = [];
        // 测试数据库中是否已经存在电子邮件地址，如果存在，则向错误数组添加验证错误
        return errors;
      },
    },
  ],
}
```

### type 

> 指定要使用的验证程序的类型。可识别的类型值如下：

- `string`：必须是类型字符串。这是默认类型

- `number`：必须是数字类型

- `boolean`：必须是布尔型

- `method`：必须是函数类型

- `regexp`：必须是regexp的实例或在创建新的Regexp时不会生成异常的字符串

- `integer`：必须是整数数字类型

- `float`：必须是浮点数字类型

- `array`：必须是由Array.isArray确定的数组

- `object`：必须是object类型，并且不是array.isArray确定的数组

- `enum`：value必须可枚举

- `date`：value必须是有效的日期类型

- `url`：必须是URL类型

- `hex`：必须是十六进制数

- `email`：必须是电子邮件类型

- `any`：可以是任何类型

### Required

`required`规则属性表示字段必须存在于正在验证的源对象上

### Pattern

`pattern`规则属性表示值必须匹配该正则表达式才能通过验证

### Range

使用`min`和`max`属性定义范围。对于字符串和数组类型是验证长度，对于数字类型，则是验证大小

### Length

要验证字段的确切长度，请指定`len`属性。对于字符串和数组类型，是验证它们的长度，而对于数字类型，此属性表示数字的精确匹配，即该数字只能严格等于len

如果len属性与min和max属性组合，len优先

### Enumerable

若要验证通过可能存在的值验证列表中的值，请使用枚举属性`enum`列出字段的有效值，例如:

```js
const descriptor = {
  role: { type: 'enum', enum: ['admin', 'user', 'guest'] },
};
```

### Whitespace

通常将只包含空格的必填字段视为错误。若要为仅由空格组成的字符串添加额外的测试，可以将`whitespace`属性设为true。规则必须是string type才有效

### Deep Rules

如果需要验证嵌套对象属性，则可以通过`rules.fields`来定义：

```js
const descriptor = {
  address: {
    type: 'object',
    required: true,
    options: { first: true },
    fields: {
      street: { type: 'string', required: true },
      city: { type: 'string', required: true },
      zip: { type: 'string', required: true, len: 8, message: 'invalid zip' },
    },
  },
  name: { type: 'string', required: true },
};

const validator = new Schema(descriptor);

validator.validate({ address: {} }, (errors, fields) => {
  // errors for address.street, address.city, address.zip
});

validator.validate({ address: {} }).catch(({ errors, fields }) => {
    // 因为开启了first: true,所有只有street和name字段的错误 
});
```

### defaultField

DefaultField 属性可以与数组或对象类型一起用于验证容器的所有值。它可以是包含验证规则的对象或数组。例如:

```js
const descriptor = {
  urls: {
    type: 'array',
    required: true,
    defaultField: { type: 'url' },
  },
};
```

### Transform

> 有时需要在验证之前转换值，可能是为了强制该值或以某种方式对其进行无害化。为此，在验证规则中添加一个`transform`函数。该属性在验证之前进行转换，并在通过验证时作为Promise结果或回调结果返回

```js
import Schema from 'async-validator';
const descriptor = {
  name: {
    type: 'string',
    required: true,
    pattern: /^[a-z]+$/,
    transform(value) {
      return value.trim();
    },
  },
};
const validator = new Schema(descriptor);
const source = { name: ' user  ' };

validator.validate(source)
  .then((data) => assert.equal(data.name, 'user'));

validator.validate(source,(errors, data)=>{
  assert.equal(data.name, 'user'));
});
```

如果没有转换函数验证将会失败，原因是parttern不匹配，因为输入包含前导空格和尾随空格，但是通过添加转换函数验证将会通过，因为在验证前字段值中的空格将被清除

### Message

将消息分配给一个规则

```js
{ name: { type: 'string', required: true, message: 'Name is required' } }
```

消息可以是任何类型，比如 jsx 格式

```js
{ name: { type: 'string', required: true, message: '<b>Name is required</b>' } }
```

Message 也可以是一个函数，使用 vue-i18n 时会用到

```js
{ name: { type: 'string', required: true, message: () => this.$t( 'name is required' ) } }
```

### asyncValidator

> 可以为指定字段自定义异步验证函数

```js
const fields = {
  asyncField: {
    asyncValidator(rule, value, callback) {
      ajax({
        url: 'xx',
        value: value,
      }).then(function(data) {
        callback();
      }, function(error) {
        callback(new Error(error));
      });
    },
  },

  promiseField: {
    asyncValidator(rule, value) {
      return ajax({
        url: 'xx',
        value: value,
      });
    },
  },
};
```

### validator

> 可以自定义指定字段的验证函数:

```js
const fields = {
  field: {
    validator(rule, value, callback) {
      return value === 'test';
    },
    message: 'Value is not equal to "test".',
  },

  field2: {
    validator(rule, value, callback) {
      return new Error(`${value} is not equal to 'test'.`);
    },
  },
 
  arrField: {
    validator(rule, value) {
      return [
        new Error('Message 1'),
        new Error('Message 2'),
      ];
    },
  },
};
```

