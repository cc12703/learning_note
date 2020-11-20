
# 流式API


## 介绍
* 流式接口是一种面向对象的API实现，用于提供更好可读性的代码。
* 使用名字级联来关联一串顺序调用指令的上下文

### 上下文
1. 通过被调用方法的返回值来定义
2. 通过自引用来传递上下文
3. 通过返回空上下文来终止


## 例子

### 非流式
```java
private void makeNormal(Customer customer) {
    Order o1 = new Order(); 
    customer.addOrder(o1); 
    OrderLine line1 = new OrderLine(6, Product.find("TAL"));   
    o1.addLine(line1); 
    OrderLine line2 = new OrderLine(5, Product.find("HPK")); 
    o1.addLine(line2); 
    OrderLine line3 = new OrderLine(3, Product.find("LGV")); 
    o1.addLine(line3); 
    line2.setSkippable(true); 
    o1.setRush(true); 
 }
```


### 流式接口
```java
private void makeFluent(Customer customer) {
        customer.newOrder()
                .with(6, "TAL")
                .with(5, "HPK").skippable()
                .with(3, "LGV")
                .priorityRush();
}
```
