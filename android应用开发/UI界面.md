

# UI界面

[TOC]

## 布局

### 分类
* LinearLayout 线性布局，布局方式为线性垂直、水平分布
* RelativeLayout 相对布局，布局方式为控件之间的相互位置关系
* FrameLayout 框架布局，所有控件都被放置在最左上的区域

### 属性

#### 公有属性

##### layout_width/height
* 设置控件的宽高

```
android:layout_width="66px"
android:layout_width="wrap_content"
android:layout_width="match_parent"
```

##### layout_margin+方位
* 控件边缘 相对于 其他控件的边距

```
android:layout_marginLeft="66px"
android:layout_marginRight="66px"
android:layout_marginTop="66px"
```

##### padding+方位
* 控件内容的边缘 相对于 控件的边距

```
android:paddingLeft="66px"
android:paddingRight="66px"
android:paddingTop="66px"
```

##### layout_gravity
* 控件 相对于 父控件的居中位置

```
android:layout_gravity="center"   //正居中
android:layout_gravity="center_horizontal"  //水平居中
android:layout_gravity="center_vertical"  //垂直居中
```

##### gravity
* 控件内容 相对于 控件的居中位置

```
android:gravity="center"   //正居中
android:gravity="center_horizontal"  //水平居中
android:gravity="center_vertical"  //垂直居中
```

#### 特有属性

##### orientation
* 用于线性布局，设置控件的排列方式

```
android:orientation="vertical"
android:orientation="horizontal"
```

##### layout_weight
* 用于线性布局，根据设置的权重，将空间按比例分配

```
android:layout_weight="1.0"
```

##### layout_alignParent + 方位
* 用于相对布局，设置控件 对齐于 父控件的某方位

```
android:layout_alignParentTop="true" //控件顶部 对齐 父控件的顶部
android:layout_alignParentLeft="true" //控件左端 对齐 父控件的左端
android:layout_centerParent="true"  //控件 位于 父控件的正中位置
android:layout_centerHorizontal="true"  //控件 位于 父控件的水平方向的中间位置
```

##### layout + 方位
* 用于相对布局，设置控件 位置 其他控件的某方位

```
android:layout_above="@id/AAA"  //控件 位于 AAA控件的上方
android:layout_below="@id/AAA"  //控件 位于 AAA控件的下方
android:layout_toLeftOf="@id/AAA"  //控件 位于 AAA控件的左方

android:layout_alignBottom="@id/AAA"  //控件的底部 对齐 AAA控件的底部
android:layout_alignLeft="@id/AAA"  //控件的左侧 对齐 AAA控件的左侧

android:layout_alignBaseline="@id/AAA"  //两个控件的基准线对齐，文本的最底下那条线

```


### 选择器
* 使控件在不同操作下显示不同的样式
* 编写 xxx_selector.xml 实现

#### 属性
* android:drawable 放置一个drawable资源
* android:state_pressed 按下状态
* android:state_focused 取得焦点状态
* android:state_selected 选中状态

```xml
//button_selector.xml

<selector xmlns:android="http://schemas.android.com/apk/res/android">

 <!-- 指定按钮按下时的图片 -->
 <item android:state_pressed="true"  
    android:drawable="@drawable/start_down"/>

 <!-- 指定按钮松开时的图片 --> 
 <item android:state_pressed="false"
    android:drawable="@drawable/start"/>

</selector>

//main.xml

<Button
  android:id="@+id/startButton"
  android:layout_width="wrap_content"
  android:layout_height="wrap_content"
  android:background="@drawable/button_selector" 
/>
```


### 形状
* 设置布局的颜色、边框线
* 编写 xxx_shape.xml 实现

```xml
//button_shape.xml

<shape xmlns:android="http://schemas.android.com/apk/res/android">
  <solid android:color="#876543"/>
   // 4个方向的边框线
  <padding
        android:bottom="0dp"
        android:left="1dp"
        android:right="1dp"
        android:top="1dp" />
   // 边框线颜色、大小
   <stroke
        android:width="1dp"
        android:color="#000000" />
   <corners android:radius="2dp" />
</shape>

//main.xml
<Button
  android:id="@+id/startButton"
  android:layout_width="wrap_content"
  android:layout_height="wrap_content"
  android:background="@drawable/button_shape" 
/>
```

#### 属性

##### corners
* 定义圆角

```
android:radius="10dp"   //全部角的圆角半径
android:topLeftRadius=”10dp"  //左上角的圆角半径
```

##### solid
* 设置内部的填充色

```
<solid android:color="color" />
```

##### stroke
* 定义描边的属性

```xml
<stroke       
    android:width="10dp"   //描边的宽度  
    android:color="color"   //描边的颜色  
    
    android:dashWidth="10dp"   //虚线的宽度，值为0时是实线  
    android:dashGap="1dp" />   //虚线的间隔
```


### 优化

#### include
* 该标签用于重用布局文件

```xml
<include layout="@layout/titlebar" />
```

##### 注意点
* 可以设置布局属性，前提要设置layout_width和layout_height
* 建议将调用布局设置宽高、位置、ID等工作放在调用布局的根标签中

#### merge
* 该标签用于在视图树中减少重复布局

```xml
//layout_1.xml
<FrameLayout>
   <include layout="@layout/layout_2"/>
</FrameLayout>

//layout_2.xml
<merge>
   <TextView />
</merge>

//合并后
<FrameLayout>
   <TextView />
</FrameLayout>
```

##### 注意点
* 必须放在布局文件的根节点上
* 并不是一个ViewGroup，也不是一个View。相当是一个占位符
* 对merge标签设置的所有属性都是无效的
* AS中预览设置 tools:parentTag="android.widget.FrameLayout"


#### ViewStub
* 是一个大小为0，默认不可见的控件
* 当被设置成可见时，布局资源才会被填充
* ViewStub本身会被填充起来的布局资源替换掉
* 被填充的布局会使用ViewStub的布局参数

```xml
<ViewStub android:id="@+id/stub"
         android:inflatedId="@+id/subTree"
         android:layout="@layout/mySubTree"
         android:layout_width="120dip"
         android:layout_height="40dip" />
```


## 屏幕适配

### 概念

#### 屏幕尺寸
* 手机对角线的物理尺寸
* 单位：英寸 （1英寸=2.54cm）


#### 屏幕分辨率
* 手机横向、纵向上的像素点总和
* 格式：横向像素点 ×　纵向像素点
* 单位：px  (1px=1像素点)


#### 屏幕像素密度
* 每英寸的像素点数
* 单位：dpi (每英寸有多少像素点)


#### 密度无关像素
* dp / dip 与终端的实际物理像素点无关

#### 独立比例像素
* sp / sip 
* 用于设置文字大小，会根据字体大小首选项进行缩放


#### 屏幕分类
| 类型 | 密度 | 典型分辨率  | dp换算 |
| --- | --- | --- | --- |
| 低密度 ldpi | 120  | 240 * 320 | 1dp=0.75px |
| 高密度 hdpi | 240  | 480 * 800 | 1dp=1.5px |
| 超高密度 xhdpi | 320 | 720 * 1280 | 1dp=2px |
| 超超高密度 xxhdpi | 480 | 1080 * 1920 | 1dp=3px |


### 方案

#### 布局匹配
* 使用相对布局，使得布局元素自适应屏幕尺寸
* 根据屏幕配置来加载对应的布局

#### 组件匹配
* 使用wrap_content、match_parent、weight来控制组件的高宽

#### 图片匹配
* 使用自动拉伸位图， xxx.9.png格式