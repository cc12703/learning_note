

## 总体

### 盒子模型

* 本质上是一个盒子，封装周围的HTML元素
* 元素包括：边距，边框，填充，和实际内容

#### 图示
![](https://gitee.com/cc12703/figurebed/raw/master/img/20201109101036.png)

#### 元素说明
* Margin（外边距）   清除边框区域。没有背景颜色，它是完全透明
* Border（边框）    边框周围的填充和内容。会受到盒子的背景颜色影响
* Padding（内边距）  清除内容周围的区域。会受到框中填充的背景颜色影响
* Content（内容）    盒子的内容，显示文本和图像


## Flex布局

* Flexible Box 弹性布局
* 使用diaplay来指定


```css
.box{
  display: flex;  //用于容器
}

.box{
  display: inline-flex;  //用于行内元素
}
```

### 基本概念

* Flex容器：采用flex布局的元素 
* Flex项目：所有子元素 
* 两根轴：水平的主轴（main） 和 垂直的交叉轴(cross)


### 图示
![](https://gitee.com/cc12703/figurebed/raw/master/img/20201109101158.png)


### 容器属性

* flex-direction 决定主轴的方向（项目的排列方向）
* flex-wrap 决定排不下时如何换行
* flex-flow direction和wrap的简写形式
* justify-content 决定项目在主轴上的对齐方式
* align-items 决定项目在交叉轴上的对齐方式
* align-content 决定项目在多根轴线上的对齐方式

#### flex-direction
* row （默认值）主轴为水平，起点在左端
* row-reverse 主轴为水平，起点在右端
* column 主轴为垂直，起点在上沿
* column-reverse 主轴为垂直，起点在下沿

##### 图示
![](https://gitee.com/cc12703/figurebed/raw/master/img/20201109101300.png)

##### 例子
```css
.box {
  flex-direction: row | row-reverse | column | column-reverse;
}
```

#### flex-wrap
* nowrap (默认值) 不换行
* wrap  换行，新行在下方
* wrap-reverse 换行，新行在上方

##### 例子
```css
.box{
  flex-wrap: nowrap | wrap | wrap-reverse;
}
```

#### flex-flow
* 简写形式
* 默认值 row nowrap

##### 例子
```css
.box {
  flex-flow: <flex-direction> || <flex-wrap>;
}
```

#### justify-content
* flex-start (默认值）左对齐
* flex-end 右对齐
* center 居中
* space-between 两端对齐，项目之间间隔相等
* space-around 每个项目两侧的间隔相等


##### 图示
![](https://gitee.com/cc12703/figurebed/raw/master/img/20201109101420.png)



##### 例子
```css
.box {
  justify-content: flex-start | flex-end | center | space-between | space-around;
}
```

#### align-items
* flex-start 交叉轴起点对齐
* flex-end 交叉轴终点对齐
* center 交叉轴中心对齐
* baseline 项的第一行文字的基线对齐
* stretch (默认值）占满整个容器的高度

##### 图示：交叉轴从上到下
![](https://gitee.com/cc12703/figurebed/raw/master/img/20201109101456.png)


##### 例子
```css
.box {
  align-items: flex-start | flex-end | center | baseline | stretch;
}
```


### 项目属性

* order 排列顺序，值越小排列越靠前，默认为0
* flex-grow 项目的放大比例，默认为0，不放大
* flex-shrink 项目的缩小比例，默认为1，空间不足将缩小
* flex-basis  项目占据的主轴空间大小
* flex 是grow, shrink, basis的缩写
* align-self 设置单个项目的对齐方式



#### flex-basis
* auto 项目本身的大小 （默认值）
* 固定值 占据固定空间



#### flex
* 快捷值 none  等于 0 0 auto
* 快捷值 auto  等于 1 1 auto



## Grid布局

* 将网页划分成一个个网格，通过任意组合不同网格来布局
* Grid是二维布局，Flex是一维布局
* 使用diaplay来指定


### 例子

```css
.box{
  display: grid;  //用于容器
}

.box{
  display: inline-grid;  //用于行内元素
}
```


### 基本概念

* 项目只能是容器的顶层子元素
* 水平区域为‘行’，垂直区域为‘列’，交叉区域为‘单元格‘


```html
<div>
  <div><p>1</p></div>
  <div><p>2</p></div>
  <div><p>3</p></div>
</div>
```

### 容器属性

* grid-template-columns 定义列，列宽
* grid-template-rows 定义行，行高


#### 例子
```css
.container {
  display: grid;
  grid-template-columns: 100px 100px 100px;
  grid-template-rows: 100px 100px 100px;
}

//使用百分比
.container {
  display: grid;
  grid-template-columns: 33.33% 33.33% 33.33%;
  grid-template-rows: 33.33% 33.33% 33.33%;
}

```