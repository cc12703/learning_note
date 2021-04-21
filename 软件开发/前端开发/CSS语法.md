[TOC]



# CSS语法

## 基础

### 总体

* CSS 含义 层叠样式表 (Cascading Style Sheets)
* 样式定义了如何显示HTML元素
* 多个样式定义可层叠合一

### 样式层叠

* 对一个元素多次设置同一个样式，将使用最后一次设置的属性值

#### 层叠优先级

**优先级从高到底**

1. 内联样式(位于HTML元素内部) 
2. 内部样式表 位于标签内）
3. 外部样式表
4. 浏览器缺省设置


## 语法

### 规则

* 格式：选择器 {声明1; 声明2; 声明3}
* 选择器用于定位HTML元素
* 声明由一个属性和一个值组成
* 可以使用注释 /* */

#### 例子
```css
    p {color:red;text-align:center;}
```


### 选择器

#### 类型
* 简单选择器
* 组合选择器（根据特定关系来选取元素）
* 伪类选择器（根据特定状态选取元素）
* 伪元素选择器
* 属性选择器 （根据属性、属性值选取元素）


#### 简单选择器
* 元素：根据元素名称来选
    ```css
    p {text-align:center;}
    ```
* id：根据元素的ID属性来选
    ```css
    #para1 {text-align:center;}
    ```
* 类：根据元素的class属性来选
    ```css
        .center {text-align:center;}
    ```
* 通用：选择所有的元素
    ```css
    * {text-align:center;}
    ```
* 分组：逗号分隔，选择所有元素
    ```css
    h1,h2,p {text-align:center;}
    ```

#### 组合选择器
* 后代元素：E 空格 F, 匹配属于E元素内的所有F元素
    ```css
    div p { background-color: yellow; }
    ```
* 子元素：E > F, 匹配E元素的所有F子元素
    ```css
    div > p { background-color: yellow; }
    ```
* 相邻兄弟元素：E + F, 匹配紧随在E元素后面的所有F元素
    ```css
    div + p { background-color: yellow; }
    ```
* 通用兄弟元素：E ~ F, 匹配E元素的所有同级的F元素
    ```css
    div ~ p { background-color: yellow; }
    ```

#### 伪类选择器
* 伪类：用于定义元素的特殊状态
* 语法：
    ```css
    selector:pseudo-class {
    property: value;
    }
    ```
* :link 选择所有未被访问的元素
    ```css
    a:link { background-color:yellow; }
    ```
* :hover 选择鼠标悬停其上的元素

#### 伪元素选择器
* ::first-line 


#### 属性选择器
* [attr]  选择带有指定属性的元素
    ```css
    a[target] { background-color: yellow; }
    ```
* [attr="value"] 选择带有指定属性和值的元素
    ```css
    a[target="_blank"] { background-color: yellow; }
    ```
* [attr~="value"] 选择属性值包含指定词的元素
    ```css
    [title~="flower"] { border: 5px solid yellow; }
    ```
* [attr|="value"] 选择属性值以指定值开头的元素（值必须是完整的单词）
    ```css
    [class|="top"] { background: yellow; }
    ```
* [attr^="value"] 选择属性值以指定值开头的元素（值不必是完整的单词）
    ```css
    [class^="top"] { background: yellow; }
    ```
* [attr$="value"] 选择属性值以指定值结尾的元素（值不必是完整的单词）
    ```css
    [class$="test"] { background: yellow; }
    ```
* [attr*="value"] 选择属性值包含有指定值的元素（值不必是完整的单词）
    ```css
    [class*="te"] { background: yellow; }
    ```

### 创建

#### 外部样式表
* 用于多个页面应用同一个样式

##### 例子
```html
<head> 
    <link rel="stylesheet" type="text/css" href="mystyle.css"> 
</head>
```

#### 内部样式表
* 用于单个文档应用特殊样式

##### 例子
```html
<head>
    <style>
        hr {color:sienna;}
        p {margin-left:20px;}
        body {background-image:url("images/back40.gif");}
    </style>
</head>
```

#### 内联样式
* 用于单个元素应用特殊样式

##### 例子
```html
<p style="color:sienna;margin-left:20px">这是一个段落。</p>
```



## 属性

### 背景
* background  简写形式，所有属性一起声明
* background-color  背景颜色   
* background-image 背景图片 
* background-size  背景大小
* background-position 背景的起始位置
* background-repeat 背景图片如何重复
* background-attachment 背景是否随其余部分滚动  

#### 例子
```css
body 
{ 
    background-image:url('img_tree.png'); 
    background-repeat:no-repeat; 
    background-position:right top; 
}

body        
{        
   background-image:url('img_tree.png');        
   background-repeat:no-repeat;        
   background-position:66% 33%;        
}
```

### 文本
* color 颜色
* line-height 行高（行间的距离）
* text-align 对齐方式 （水平对齐方式）
* text-decoration 修饰
* text-transform  转换 （大写，小写）

* font-family 字体系列
* font-style 字体样式  （正常、斜体、倾斜）
* font-size 字体大小  （绝对大小、相对大小）

#### line-height
* normal 默认值
* number  值 = number * 当前的字体尺寸
* length  固体值
* %       当前字体尺寸的百分比
* inherit  继承父元素的值


## 值

### 单位
* %   百分比
* em  当前字体尺寸
* px  像素


### 颜色
* 十六进制：#RRGGBB
* RGB:  rgb(red, green, blue)
* RGBA: rgba(red, green, blue, alpha)
* 预定义名：black, blue, gray


## 选择符



## 定位

### 机制

* 包括：普通流、浮动、绝对定位

#### 普通流
* 元素的显示位置由其在html中的位置决定
* 块级框从上到下一个接一个排列
* 行内框在一行中水平布置


### 定位属性

* position 元素需要放置的位置
* top  元素上外边距与块上边界的偏移
* right 元素右外边距与块右边界的偏移
* overflow 元素内容溢出时的处理
* z-index 元素的堆叠顺序


#### position

* position属性用来指定一个元素在网页上的位置

##### 可选值
* static  默认值
* relative
* fixed
* absolute
* sticky

##### static 
* 浏览器会自主决定每个元素的位置
* 每个块级元素会占据自己的区块，元素间不会产生重叠
* 正常页面流：浏览器按照源码顺序，决定每个元素的位置

**图示**
![](https://gitee.com/cc12703/figurebed/raw/master/img/20201109102131.png)


##### relative
* 相对于默认位置(static时的位置）进行偏移
* 必须使用top, bottom, left, right来指定偏移的方向和距离

**图示**
![](https://gitee.com/cc12703/figurebed/raw/master/img/20201109102153.png)


**示例**
```css
div {
  position: relative;
  top: 20px;
}
```

##### absolute
* 相对于父元素进行偏移，定位基点：父元素
* 定位基点不能是static的
* 必须使用top, bottom, left, right来指定偏移的方向和距离
* 该元素会被“正常页面流”忽略，所占空间为零


**图示**
![](https://gitee.com/cc12703/figurebed/raw/master/img/20201109102215.png)


##### fixed
* 相对于浏览器窗口进行偏移，定位基点：浏览器窗口
* 导致元素位置不随页面滚动而变化
* 必须使用top, bottom, left, right来指定偏移的方向和距离


**图示**
![](https://gitee.com/cc12703/figurebed/raw/master/img/20201109102235.png)



##### sticky
* 会产生动态效果
* 会在relative和fixed之间切换
* 需要搭配top, bottom, left, right来一起使用

**示例**
```css
#toolbar {
  position: -webkit-sticky; /* safari 浏览器 */
  position: sticky; /* 其他浏览器 */
  top: 20px;
}
```

**规则**
* 初始加载时时默认定位 relative
* 当页面滚动，父元素脱离窗口
* 只要与sticky元素的距离达到生效门槛，定位会自动切换为fixed
* 等到父元素完全脱离窗口时，定位又会切换回relative


#### z-index

* 值高元素总是在值低元素的前面

##### 可选值
* auto  默认，与父元素一样
* number 顺序值