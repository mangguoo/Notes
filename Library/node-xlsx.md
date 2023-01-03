#### node-xlsx

> 创建并写入xlsx

``` js
import xlsx from 'node-xlsx'
import fsp from 'fs/promises'
import path from 'path'

const data = [
  [1, 2, 3],
  [true, false, null, 'sheetjs'],
  ['foo', 'bar', new Date('2014-02-19T14:30Z'), '0.3'],
  ['baz', null, 'qux']
]

;(async function init() {
  const buffer = xlsx.build([{ name: 'mySheetName', data }])
  await fsp.writeFile(path.join(path.resolve(), 'a.xlsx'), buffer)
})()
```

> 读取并筛选xlsx

``` js
import xlsx from 'node-xlsx'
import fsp from 'fs/promises'
import path from 'path'

;(async function init() {
  let data = await fsp.readFile(path.join(path.resolve(), 'abc.xlsx'))
  data = xlsx.parse(data)[0] // 获取第一张sheet
  const arr = data.data.slice(0, 4).concat(data.data.slice(4).filter((item) => item[5] >= 90)) // 不筛选表头
  const buffer = xlsx.build([{ name: 'GP27考试90分以上的表格', data: arr }])
  await fsp.writeFile(path.join(path.resolve(), '90Up.xlsx'), buffer)
})()
```

