


# tsc


## tsconfig.json

### 总配置项

* compileOnSave
* compileOptions
* include
* exclude
* files
* extends
* references

### compileOnSave

* 功能：保存文件时是否自动编译


### compileOptions

* 功能：配置编译选项

#### 常用配置

* incremental 是否启用增量编译
* target 编译出的目标语言版本
* module 生成代码的模板标准
* allowJS 是否允许编译js,jsx文件

* moduleResolution 模块解析策略，默认为node(相对方式导入)
* baseUrl 解析非相对模块的基地址，默认为当前目录
* paths  路径映射，相对于baseUrl

* declaration 是否生成声明文件
* declarationDir 生成的声明文件存放目录

* typeRoots 声明文件目录，默认值为node_modules/@types
* types 加载的声明文件包

* strict 是否开启所有严格的类型检查
* importHelpers 是否通过tslib引入helper函数
* esModuleInterop 是否允许export导出，import from导入


### files

* 功能：指定要编译的单个文件列表

```json
"files": [
  // 指定编译文件是src目录下的a.ts文件
  "scr/a.ts"
]
```

### include

* 功能：指定要编译的文件，目录

```json
"include": [
  // "scr" // 会编译src目录下的所有文件，包括子目录
  // "scr/*" // 只会编译scr一级目录下的文件
  "scr/*/*" // 只会编译scr二级目录下的文件
]
```

### exclude

* 功能：指定要排除编译的文件，目录
* 默认：排除node_modules文件夹

```json
"exclude": [
  // 排除src目录下的lib文件夹下的文件不会编译
  "src/lib"
]
```

### extends

* 功能：引入其他配置文件，继承配置

```json
// 把基础配置抽离成tsconfig.base.json文件，然后引入
"extends": "./tsconfig.base.json"
```