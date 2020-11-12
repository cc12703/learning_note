

# xpath语法


## 选取节点

* /  从根节点选取
* // 不考虑节点的位置
* **.**  选取当前节点
* **..** 选取当前节点的父节点
* @    选取属性
* **[ ]**  谓语，用于查找特定的节点
* \*  通配符，用于匹配所有节点
* @*  通配符，用于匹配所有属性节点


### 例子

* bookstore     选取bookstore元素下的所有子节点          
* /bookstore    选择根元素bookstore
* //book        选择所有的book节点
* //@lang       选取名为lang的所有属性
* /bookstore/book[1]    选取bookstore子节点的第一个book节点
* /bookstore/book[last()] 选取bookstore子节点的最后一个book节点
* //title[@lang]     选取所有拥有lang属性的title节点
* //title[@lang='eng']     选取所有lang属性值为eng的title节点
* /bookstore/book[price>35.00]/title  选取bookstore节点中book节点下的所有title节点, 并且price属性的值必须大于35.00
* /bookstore/*      选取bookstore节点下的所有子节点
* **//***          选取文档中的所有元素
* **.//***        选取当前节点所包含的所有元素
* //title[@*]      选取所有带有属性的title节点