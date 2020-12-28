
# markmap

[TOC]

## 功能
* 使用思维脑图，可视化markdown文件

### 原理
* 将markdown解析成节点树，保存入html中
* html显示时，将节点树动态渲染成思维脑图

## 模块
* markmap-lib 将markdown文件转换成html文件
* markmap-view 在浏览器中将html文件渲染成脑图
* markmap-toolbar 在浏览器中显示浮动菜单栏
* markmap-cli 命令行工具

## 数据结构

### 节点树
* 由INode节点构成的树

#### INode属性
* d 节点深度
* v 内容值
* t 类型
* c 子节点列表
* p 运行时信息（渲染时用）

##### 类型
* heading 标题
* list_item 列表项
* fence 已渲染的外部内容
* bullet_list 列表

##### 运行时信息
* i   唯一ID
* el  html内容
* s   数组：宽，高 信息
* f   是否显示标识

### html文件
```html
<head>
<style></style>  <!-- 样式 -->
<link rel="stylesheet" href="xxx"> <!-- 外部样式 -->
</head>

<body>
<svg id="mindmap"> </svg>  <!-- 思维脑图的根节点 -->
<script src="xxx">    <!-- 渲染需要用到的代码 -->

<!-- 触发渲染 -->
<!-- data为节点树的序列化字符串 -->
<script> 
window.mm = Markmap.create('svg#mindmap', null, data)
</script>

</body>
```

### svg图
```html
<svg>
  <g> <!-- 根节点 -->
    <path>  <!-- 线条 -->
    ...

    <g>  <!-- 内部节点 -->
        <rect>  <!-- 线条 -->
        <circle>  <!-- 节点圆圈 -->
        <foreignObject> <!-- 内容对象 -->
    </g>
    ...
</svg>
```


## 功能

### 文件转换

#### 流程
1. 用transform 解析markdown文件为节点树
    1. 使用remarkable库解析markdown文件
    1. 用buildTree 生成节点树
1. 用fillTemplate 将节点树持久化到html文件
    1. 用persistCSS 生成CSS的html代码
    1. 用persistJS  生成JS的html代码
    1. 节点树被序列化成字符串存入html中
    1. 将html代码替换进模板，生成html文件


### 渲染

#### 流程
1. 完善节点树信息
1. 使用d3-flextree生成svg的节点树