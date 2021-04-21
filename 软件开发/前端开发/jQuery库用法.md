

# jQuery库用法


[TOC]


## 概述

### 功能
* 获取文档中的元素：通过选择符机制，简化了在DOM中选择元素的工作
* 修改页面的外观：提供了跨浏览器的解决方案
* 修改页面的内容
* 响应用户的交互操作：截获各种页面事件
* 为页面添加动态效果
* 无需刷新页面从服务器获取信息：使用Ajax

### 特点
* 查找页面元素的机制构建于CSS选择符
* 通过创建插件来支持扩展
* 抽象了浏览器的不一致性
* 总是面向集合：使用隐式迭代技术，去除循环结构
* 通过链式操作将多重操作集于一行


## 选择元素

### $()函数
* 参数为CSS选择符，返回包含了对应元素的jQuery对象

### 基本选择符
* 标签名:  $('p')  取得文档中所有的段落
* ID: $('#some-id') 取得文档中ID为some-id的一个元素
* 类: $('.some-class') 取得文档中类为some-class的所有元素

### CSS选择符
* $('#selected-plays > li') 
    * 查找ID为selected-plays元素的子元素(>)中所有的列表项
* $('#selected-plays li:not(.horizontal)') 
    * 查找ID为selected-plays元素的后代元素中没有horizontal类的元素
* $('a[href^="mailto:"]') 
    * 查找所有带href属性的，且以mailto开头的a元素
* $('a[href$=".pdf"]') 
    * 查找所有带href属性的，且以.pdf结尾的a元素
* $('a[href^="http"][href*="henry"]')
    * 查找所有带href属性的，且以http开头，且任意位置包含henry的a元素

### 自定义选择符
* 选择符以冒号开头
* $('div.horizontal:eq(1)')
    * 获取所有带horizontal类的div中的第2个元素（js数组从0开始）
* $('tr:even')
    * 获取所有的偶数行，按所有元素来计算位置
* $('tr:odd')
    * 获取所有的奇数行，按所有元素来计算位置
* $('tr:nth-child(odd)')
    * 获取所有的奇数行，从父元素开始计算位置
* $('td:contains(Henry)')
    * 获取所有内容中包含Henry的列元素，区分大小写


### 基于表单的选择符
* :input 输入元素
* :button 按钮元素、属性值为button的输入元素
* :enabled 启用的表单元素
* :checked 已勾选的单选按钮或复选框
* :selected 被选中的选项元素



## 事件

### 处理事件
* on() 给指定DOM事件，注册一个方法
    ```js
    $('#switcher-large').on('click', function() {
        $('body').addClass('large')
    })
    ```

#### 简化写法
* click事件简化成click()
    ```js
    $('#switcher-large').click(function() {
        $('body').addClass('large')
    })
    ```

### 事件传播
* 事件捕获
* 事件冒泡
