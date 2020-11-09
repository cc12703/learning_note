

# HTML5语法


## 改进点

* 增加了新元素、新属性
* 完全支持CSS3
* 支持video, audio
* 支持 2D/3D制图
* 支持本地存储
* 支持本地SQL数据

## 最小的html5文档
```html
<!DOCTYPE html>
            
<html>
    <head>
        <title>文档标题</title>
    </head>
                
	<body>          
	  文档内容......   
	</body>
                
</html>
```

## 元素

### canvas
* 一个图形容器，使用脚本来绘制图形

```html
<canvas id="myCanvas" width="200" height="100" 
	style="border:1px solid #000000;"> 
</canvas> 

<!-- 绘制 -->
<script> 
	var c=document.getElementById("myCanvas"); 
	var ctx=c.getContext("2d"); 
	ctx.fillStyle="#FF0000"; 
	ctx.fillRect(0,0,150,75); 
</script>
```

#### 绘制

##### 坐标
* 左上角的坐标为 (0,0)

##### 路径
* moveTo() 定义线条的开始坐标
* lineTo() 定义线条的结束坐标

##### 文本
* font 定义字体
* fillText()  绘制实心文本


#### 特点

* 依赖分辨率
* 不支持事件处理器
* 文本渲染能力弱
* 能够包图像保存成png，jpg图片
* 适合于图像密集型的游戏应用


### 内嵌SVG

* 直接定义SVG图形

```html
<svg xmlns="http://www.w3.org/2000/svg" version="1.1" height="190">
	  <polygon points="100,10 40,180 190,60 10,60 160,180"
	  style="fill:lime;stroke:purple;stroke-width:5;fill-rule:evenodd;">
</svg>
```

#### 特点
* 不依赖分辨率
* 支持事件处理器
* 复杂度高会减慢渲染速度
* 适合于带有大型渲染区域的应用程序

