

# Java8新语法



## Lambda表达式

### 语法
* ( parameters ) -> expression
* ( parameters ) -> { statements; }


### 特点
* 不需要声明参数类型
* 一个参数时无需定义园括号，多个参数时才需要
* 一个语句时无需使用大括号
* 只有一个表达式时，编译器会自动返回值
* 只能引用final标记的外层局部变量



## 方法引用

* 通过方法名来指向一个方法
* 使用一对冒号 **::**

### 语法
* 构造器，无参数 Class::new，Class<T>::new  
* 静态方法，一个类型参数 Class::static_method
* 特定类的任意对象的方法，无参数 Class::method
* 特定对象的方法，一个类型参数  Instance::method


## Stream库

### 组成
* 数据源：生成流对象
* 中间操作：用于对数据进行处理
* 最终操作：得到最终结果

### 特点
* pipeline： 中间操作会返回流对象本身，用于串联多个操作
* 内部迭代：迭代处理在流内部进行

### 操作符

#### map

* 中间操作，将输入元素映射为输出元素

```java
    wordList.stream()
          .map(String::toUpperCase)
          .collect(Collectors.toList());
```

#### flatmap

* 中间操作，将输入元素进行结构扁平化

```java
Stream<List<Integer>> inputStream = Stream.of(
        Arrays.asList(1),
        Arrays.asList(2, 3),
        Arrays.asList(4, 5, 6)
    );
Stream<Integer> outputStream = inputStream
        .flatMap((childList) -> childList.stream());
```

#### filter

* 中间操作，对输入元素进行测试，通过测试的元素被输出

```java
Integer[] sixNums = {1, 2, 3, 4, 5, 6};
Integer[] evens = Stream.of(sixNums)
            .filter(n -> n%2 == 0)
            .toArray(Integer[]::new);
```

#### sorted

* 中间操作，对输入元素进行排序

```java
List<Person> personList2 = persons.stream()
        .sorted((p1, p2) -> p1.getName().compareTo(p2.getName()))
        .collect(Collectors.toList());
```


#### forEach

* 最终操作，对每个元素执行lambda表达式

```java
roster.stream()
    .filter(p -> p.getGender() == Person.Sex.MALE)
    .forEach(p -> System.out.println(p.getName()));
```

#### reduce

* 最终操作，将输入元素组合起来。提供一个起始值和运算规则

```java
Integer sum = integers.reduce(0, (a, b) -> a+b);
Integer sum = integers.reduce(0, Integer::sum);
```


#### 其他
* limit ：中间操作，返回前面n个元素
* skip  ：中间操作，扔掉前面n个元素
* distinct ：中间操作，去重输入元素
* findFirst ： 最终操作，返回第一个元素，Optional类型
* allMatch ： 流中的所有元素都符合条件，才返回true
* anyMatch ： 流中只要有一个元素符合条件，就返回true
* noneMatch ： 流中没有一个元素符合条件，才返回true


## Optional类

### 特点
* 一个容器，可以保存类型T的值，或仅仅保存null值
* 用于解决空指针异常

### 方法
* get() ：返回值，或异常NoSuchElementException
* isPresent() ： 如果值存在，返回true
* orElse(other) ：如果值存在，返回值。不存在返回other


## android兼容使用

* lambda、方法应用：使用retrolambda插件
* stream库：使用Lightweight-Stream-API库


### stream库

#### filterNot
```java
// Java 8
stream.filter(((Predicate<String>) String::isEmpty).negate())
// LSA
stream.filterNot(String::isEmpty)
```

#### withoutNulls
* 过滤掉空对象

```java
Stream.of("a", null, "c", "d", null)
    .withoutNulls() // [a, c, d]
```

