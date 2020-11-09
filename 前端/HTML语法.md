
# HTML语法

## 基础

* 超文本标记语言 Hyper Text Markup Language
* HTML 是一套标记标签 ，用来描述网页
* HTML 文档包含了HTML 标签及文本内容

### 网页结构
```html
<html>
	<body>
		<h1> xxxx </h1>
		<p>  xxxx </p>
		<p>  xxxx </p>
	</body>
</html>
```

## 元素

### 公共属性
* class   定义元素的一个或多个类名
* id       定义元素的唯一ID
* style   规定元素的行内样式
* title    描述了元素的额外信息 

### 标题
* 使用 h1 - h6 标签进行定义的
* h1定义最大的标题。h6 定义最小的标题

```html
<h1>这是一个标题。</h1> 
<h2>这是一个标题。</h2> 
<h3>这是一个标题。</h3> 
```

#### 注意
* 浏览器会自动地在标题的前后添加空行
* 搜索引擎会使用标题给网页编制索引


### 水平线
* 使用 hr 标签进行定义

```html
<p>这是一个段落。</p>
<hr>
<p>这是一个段落。</p>
<hr>
<p>这是一个段落。</p>
```

### 段落
* 使用 p 标签进行定义

#### 注意
* 浏览器会自动地在段落的前后添加空行

### 换行
* 使用 br 标签进行定义

```html
<p>这个<br>段落<br>演示了分行的效果</p>
```

### 超链接
* 使用 a 标签进行定义
* href属性为链接的地址

```html
<a href="//www.w3cschool.cn/">Visit W3CSchool</a>

<!-- 空链接
<a href="#">W3CSchool</a>  
```

### 列表

#### 无序
```html
<ul>
   <li>Coffee</li>
   <li>Milk</li>
 </ul> 
```

#### 有序
```html
<ol>
 <li>Coffee</li>
 <li>Milk</li>
</ol> 
```

#### 自定义
```html
<dl>   
 <dt>Coffee</dt>     <!-- 自定义列表项目 -->
	 <dd>- black hot drink</dd>  <!-- 自定义列表的描述 -->
 <dt>Milk</dt>
	 <dd>- white cold drink</dd>
</dl> 
```

### 表格
* table 标签定义表格
* th 标签定义标题栏
* tr 标签定义行
* td 标签定义列

```html
<table border="1">
	<tr>
		<th>Header 1</th>
		<th>Header 2</th>
	</tr>
	<tr>
		<td>row 1, cell 1</td>
		<td>row 1, cell 2</td>
	</tr>
	<tr>
		<td>row 2, cell 1</td>
		<td>row 2, cell 2</td>
	</tr>
</table>
```


### 区块
* 用于将HTML元素组合起来

#### div
* 可以把文档分割为独立的、不同的部分
* 是块级元素，是可用于组合其他 HTML 元素的容器
* 属于块级元素，浏览器会在其前后显示折行

#### span
* 是内联元素，可用作文本的容器

```html
<p>我的母亲有 <span style="color:blue">蓝色</span> 的眼睛。</p>
```

### 表单
* 表单是一个包含表单元素的区域，表单元素是允许用户在表单中输入内

```html
<form>
.
input elements
.
</form>
```

#### 输入元素

##### 文本域
```html
<form>
	First name: <input type="text" name="firstname"><br>
	Last name: <input type="text" name="lastname">
</form> 
```

##### 密码字段
```html
<form>
	Password: <input type="password" name="pwd">
</form> 
```

##### 提交按钮
* 当用户单击确认按钮时，表单的内容会被传送到另一个文件

```html
<form name="input" action="html_form_action.php" method="get">
	Username: <input type="text" name="user">
	<input type="submit" value="Submit">
</form> 
```

### 框架

* iframe 标签规定一个内联框架
* 一个内联框架被用来在当前 HTML 文档中嵌入另一个文档

```html
<iframe src="demo_iframe.htm" width="200" height="200"></iframe>
```


## 布局
* 大多数网站会把内容安排到多个列中
* 大多数网站可以使用 div 或者 table 元素来创建多列
* table标签不建议作为布局工具使用的 - 表格不是布局工具

```html
<!DOCTYPE html>
<html>
	<body>

		<div id="container" style="width:800px">

		<div id="header" style="background-color:#FFA500;">
			<h1 style="margin-bottom:0;">Main Title of Web Page</h1></div>

		<div id="menu" style="background-color:#FFD700;height:200px;width:100px;float:left;">
			<b>Menu</b><br>
			HTML<br>
			CSS<br>
			JavaScript</div>

		<div id="content" style="background-color:#EEEEEE;height:200px;width:700px;float:left;">
			Content goes here</div>

		<div id="footer" style="background-color:#FFA500;clear:both;text-align:center;">
				Copyright © w3cschool.cn</div>

		</div>
 
	</body>
</html>
```

## XHTML

* 以XML格式来编写的HTML
* XHTML 指的是可扩展超文本标记语言
* XHTML 是更严格更纯净的 HTML 版本
* XHTML 是大小写敏感的，标签应该使用小写

### 与HTML区别

* DOCTYPE 是强制的
* html标签中的namespace 属性是强制的
* html, head, title, body 标签也是强制的
* 元素和属性必须使用小写


## HTML媒体

### 插件

* 用于扩展HTML浏览器的功能，可用于播放视频和音频

#### object元素
* 定义了在HTML中嵌入的对象

```html
<object width="400" height="50" data="bookmark.swf"></object>
```
