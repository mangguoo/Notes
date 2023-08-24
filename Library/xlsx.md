### js-xlsx

>js-xlsx 提供的接口非常清晰主要分为两类：
>
>- xlsx对象本身提供的功能
>   - 解析数据
>- utils工具类
>   - 将数据添加到数据表对象上
>   - 将二维数组以及符合格式的对象或者HTML转为工作表对象
>   - 将工作簿转为另外一种数据格式
>   - 行,列,范围之间的转码和解码
>   - 工作簿操作
>   - 单元格操作

#### 常用API解析：

| 名称                                            | 作用                            | 示例                                                        |
| ----------------------------------------------- | ------------------------------- | ----------------------------------------------------------- |
| read(data, {type: options})                     | 读取一个excel文件并返回book对象 | XLSX.read(data, {type: options});                           |
| book_new()                                      | 创建一个book对象                | const wb= utils.book_new();                                 |
| sheet_to_json(Sheet)                            | 将一个sheet对象转为JSON对象     | const sheetObj = utils.sheet_to_json(wb.Sheets[sheetName]); |
| json_to_sheet(data)                             | 将一个JSON对象数组转为sheet对象 | const sheet = utils.json_to_sheet(this.studentList);        |
| aoa_to_sheet(data)                              | 将一个数组对象转为sheet对象     | const sheet = utils.aoa_to_sheet(this.studentList);         |
| book_append_sheet(workbook, sheet, “sheet名称”) | 将sheet对象添加到book中         | utils.book_append_sheet(wb, sheet, "sheetName");            |

#### 深入了解`WorkBook`对象和`Sheet`对象

**`WorkBook`对象结构：**

> 通过 xlsx.read(data, { type: option }); 得到一个workbook对象
> 这里option的类型有以下几个：
>
> - base64: 以base64方式读取
>
> - binary: BinaryString格式(byte n is data.charCodeAt(n))
>
> - string: UTF8编码的字符串
>
> - buffer: nodejs Buffer
>
> - array: Uint8Array，8位无符号数组
>
> - file: 文件的路径（仅nodejs下支持）
>
> 通过read()函数读取到的workbook对象及他的结构：
>
> ![36098d2ed0e5973b1e26b05933fc2735](https://cdn.jsdelivr.net/gh/ilmangoi/imgRepo@main/img/36098d2ed0e5973b1e26b05933fc2735.png)
>
> 其中主要看`Props、SheetNames、Sheets`这三个属性
>
> - `Props`：存储文档的基本信息，例如文件作者、文件创建时间、最后修改时间、最后修改作者等等信息
> - `SheetNames`：存储工作簿中所有工作表的名称，为数组形式
> - `Sheets`：存储工作簿中所有工作表的详细数据

**Sheet 对象结构：**

> `Sheets`保存了每个sheet的具体内容（我们称之为`Sheet Object`）。每一个`sheet`是通过类似`A1`这样的键值保存每个单元格的内容，我们称之为单元格对象（`Cell Object`），每一个`Sheet Object`表示一张表格，只要不是`!`开头的都表示普通`cell`
>
> ![08631ba46493789893aeecc38a7055a7](https://cdn.jsdelivr.net/gh/ilmangoi/imgRepo@main/img/08631ba46493789893aeecc38a7055a7.png)
>
> - `!ref`：表示该表中数据所占用的单元格范围
> - `!margins`：存储单元格边框信息
> - `!merges`：存放一些单元格合并信息，是一个数组，每个数组由包含`s`和`e`构成的对象组成，`s`表示开始，`e`表示结束，`r`表示行，`c`表示列

**Cell 对象结构：**

> 每一个单元格都是一个对象（`Cell Object`），它主要有`t`、`v`、`r`、`h`、`w`等字段：
>
> - t：表示内容类型
>   - `s`表示string类型
>   - `n`表示number类型
>   - `b`表示boolean类型
>   - `d`表示date类型
>   - …
> - v：表示原始值
> - f：表示公式，如`B2+B3`
> - h：HTML内容
> - w：格式化后的内容
> - r：富文本内容`rich text`

#### 写存xlsx文件：

- /config/site.js

```js
export default {
  exportUserFieldName: {
    id: '序号',
    name: '姓名',
    age: '年龄',
    sex: '性别[1:先生,2:女士]',
    address: '地址'
  },
}
```

- /src/utils/tools.js

> setJsonFieldToExcelField是为了把data中的key映射成汉字
>
> type参数为表示要把配置文件的key与value反转，这是因为如果是导入excel文件的话，那么它的field是汉字，因此要把这些field再映射回字母

```js
export const setJsonFieldToExcelField = (cnField, data, type = 1) => {
  if (type == 0) {
    let fieldKeys = Object.keys(cnField)
    cnField = Object.values(cnField).reduce((prev, curr, index) => {
      prev[curr] = fieldKeys[index]
      return prev
    }, {})
  }
  return data.map(item => {
    let keys = Object.keys(item)
    let obj = {}
    keys.forEach(key => {
      cnField[key] ? obj[cnField[key]] = item[key] : null
    })
    return obj
  })
}

export const fileToBinary = (fileObj, type = 'binary') => {
  return new Promise((resolve, reject) => {
    let read = new FileReader()
    if (type === 'binary') {
      // 读取的过程是一个异步的
      read.readAsBinaryString(fileObj)
    } else {
      read.readAsArrayBuffer(fileObj)
    }
    read.onload = evt => {
      // 把文件对象读成binary
      resolve(evt.target.result)
    }
  })
}
```

- /src/views/userList.vue

```js
import { setJsonFieldToExcelField } from "@/utils/tools";
import siteConfig from "@/config/site";
import { writeFile, utils } from "xlsx";

export default {
    exportExcel() {
      // 生成对应的excel文件
      let data = setJsonFieldToExcelField(siteConfig.exportUserFieldName, this.tableData);
      // json转为sheet工作单元
      let sheet = utils.json_to_sheet(data);
      // 工作薄
      let workBook = utils.book_new();
      // 工作单元添加到工作薄中
      utils.book_append_sheet(workBook, sheet, "用户管理");
      // 生成xlsx
      writeFile(workBook, "用户管理.xlsx");
    },
  },
};
```

#### 读取xlsx文件：

> 读取xlsx表格，并进行展示

```vue
<template>
  <div style="display: flex">
    <el-upload action="" :auto-upload="false" :show-file-list="false" :on-change="changeFile">
      <el-button type="success">上传excel文件</el-button>
    </el-upload>
  </div>
  <el-divider />
  <el-table :data="userExcelData">
    <el-table-column type="index" label="#"></el-table-column>
    <el-table-column prop="name" label="姓名"></el-table-column>
    <el-table-column prop="age" label="年龄"></el-table-column>
  </el-table>
</template>

<script>
import { utils, read } from "xlsx";
import { fileToBinary, setJsonFieldToExcelField } from "@/utils/tools";
import siteConfig from "@/config/site";
export default {
  data() {
    return {
      userExcelData: [],
    };
  },
  methods: {
    async changeFile(file) {
      if (!file) return;
      // 把得到的File对象，转为 binary 数据源  ==> 在html5中提供 FileReader对象
      let binaryString = await fileToBinary(file.raw);
      // 使用xlsx库完成对于数据源的解析
      let workBook = read(binaryString, { type: "binary" });
      // 读取对应单元薄中的数据
      let sheet = workBook.Sheets[workBook.SheetNames[0]];
      let json = utils.sheet_to_json(sheet);
      this.userExcelData = setJsonFieldToExcelField(siteConfig.exportUserFieldName, json, 0);
    },
  },
};
</script>
```

#### 操作合并单元格：

```vue
<template>
  <div>
    <el-button type="primary" @click="exportExcel">导出三年（2）班学生信息</el-button>
  </div>
</template>

<script>
import { writeFile, utils } from "xlsx";

export default {
  name: "Demo",
  data() {
    return {
      studentList2: [
        ["三年二班学生信息"],
        ["姓名", "年龄", "性别"],
        ["小明", 9, "男"],
        ["小红", 8, "男"],
        ["小智", 9, "男"],
        ["小秀", 9, "女"],
        ["小林", 8, "性别"],
        ["小晨", 9, "女"],
      ],
    };
  },
  methods: {
    exportExcel() {
      // 创建一个book对象
      const wb = utils.book_new();
      // 将学生数组对象转为sheet表
      // aoa_to_sheet 是将数组转为sheet
      const sheet = utils.aoa_to_sheet(this.studentList2);
      // 设置合并单元格区间
      sheet["!merges"] = [
        { s: { r: 0, c: 0 }, e: { r: 0, c: 2 } }, // 合并A1:C1
      ];
      // 将sheet 添加到book中
      utils.book_append_sheet(wb, sheet, "sheetName");
      // 导出xlsx
      writeFile(wb, `三年二班学生信息.xlsx`);
    },
  },
};
</script>
```

