

# Java常用语法


## 数据类型


### 相互转换

#### array to list
```java
    String[] array=new String[3]; 
    List<String> list=Arrays.asList(array);
```

#### list to array
```java
     List<String> list=new ArrayList<String>();
     String[] array = (String[])list.toArray(new String[list.size()]); 
```


## 编码

### 编译时的编码
* 不指定，javac将按系统默认的编码来转换成unicode，即 System.getProperty("file.encoding") 的值
* 可以指定 javac -encoding utf-8


### 运行时的编码
* 不指定，System.out.println将按系统默认的编码来转换
* 可以指定  java -Dfile.encoding=utf8


### dom4j编码

#### 读操作
```java
	SAXReader saxReader = new SAXReader(); 
    saxReader.setEncoding("UTF8");
    saxReader.read(new File(fileName));
```

#### 写操作
```java
     OutputFormat format =  OutputFormat.createPrettyPrint();
     format.setEncoding("UTF8");
     
     //一定要使用Stream
     OutputStream  outStream = new FileOutputStream(new File(outFile));
     XMLWriter output = new XMLWriter(outStream, format);
     output.write(doc);
     output.close();
```


## 其他

### jar包中读取文件
*  使用 this.getClass.getResourceAsStream(/xxx/xxx)  接口
*  / 开头的从class的输出根目录开始
*  ./ 开头的从当前类所在的包名目录开始


### 反射调用

#### forName获取数组类型
 * byte[].class  使用参数  [B 
 * byte[][].class 使用参数   [[B]]
 * String[].class 使用参数  [Ljava.lang.String;
 