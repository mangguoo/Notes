# chalk

<img src="https://raw.githubusercontent.com/ilmangoi/imgRepo/main/img/screenshotfjskldjfalks;djfas.png" alt="screenshotfjskldjfalks;djfas" style="zoom: 67%;" />

> ### chalk.`<style>[.<style>...](string, [string...])`
>
> 例子：`chalk.red.bold.underline('Hello', 'world');`
>
> 链式调用样式并将最后一个样式作为带有字符串参数的方法调用。顺序无关紧要，如果发生冲突，以后的样式优先
>
> 这意味着`chalk.red.yellow.green`等同于`chalk.green`，多个参数将以空格分隔

```js
import chalk from 'chalk';

console.log(chalk.blue('Hello world!'));
```

```js
import chalk from 'chalk';

const log = console.log;

// Combine styled and normal strings
log(chalk.blue('Hello') + ' World' + chalk.red('!'));

// Compose multiple styles using the chainable API
log(chalk.blue.bgRed.bold('Hello world!'));

// Pass in multiple arguments
log(chalk.blue('Hello', 'World!', 'Foo', 'bar', 'biz', 'baz'));

// Nest styles
log(chalk.red('Hello', chalk.underline.bgBlue('world') + '!'));

// Nest styles of the same type even (color, underline, background)
log(chalk.green(
	'I am a green line ' +
	chalk.blue.underline.bold('with a blue substring') +
	' that becomes green again!'
));

// ES2015 template literal
log(`
  CPU: ${chalk.red('90%')}
  RAM: ${chalk.green('40%')}
  DISK: ${chalk.yellow('70%')}
`);

// Use RGB colors in terminal emulators that support it.
log(chalk.rgb(123, 45, 67).underline('Underlined reddish color'));
log(chalk.hex('#DEADED').bold('Bold gray!'));
```

自定义打印样式

```js
import chalk from 'chalk';

const error = chalk.bold.red;
const warning = chalk.hex('#FFA500'); // Orange color

console.log(error('Error!'));
console.log(warning('Warning!'));
```

利用console.log中的字符串替换

```js
import chalk from 'chalk';

const name = 'Sindre';
console.log(chalk.green('Hello %s'), name);
// => 'Hello Sindre'
```

