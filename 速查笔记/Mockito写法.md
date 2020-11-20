
# Mockito写法



## 模拟行为
```java
//You can mock concrete classes, not just interfaces
 LinkedList mockedList = mock(LinkedList.class);

 //stubbing
 when(mockedList.get(0)).thenReturn("first");
 when(mockedList.get(1)).thenThrow(new RuntimeException());
```


### 模拟抛出异常
```java
doThrow(new RuntimeException())
        .when(mockedList).clear();
```

### 模拟多次不同行为
**超过模拟次数，将只返回最后一次的值**
```java
when(mock.someMethod("some arg"))
   .thenThrow(new RuntimeException())
   .thenReturn("foo");

 when(mock.someMethod("some arg"))
   .thenReturn("one", "two", "three");
```

### 使用回调模拟



## 校验行为

```java
//mock creation
 List mockedList = mock(List.class);

 //using mock object
 mockedList.add("one");
 mockedList.clear();

 //verification
 verify(mockedList).add("one");
 verify(mockedList).clear();
```


### 使用参数匹配
**如果使用了参数匹配器，所有的参数都需要使用**

```java
//you can also verify using an argument matcher
verify(mockedList).get(anyInt());

//correct
verify(mock).someMethod(anyInt(), anyString(), eq("third argument"));

//incorrect
verify(mock).someMethod(anyInt(), anyString(), "third argument");
```


### 校验调用次数
* never()      没有调用过
* times()      固定次数
* atLeast()   最少次数
* atMost()   最多次数

```java
//exact number of invocations verification
 verify(mockedList, times(2)).add("twice");
 verify(mockedList, times(3)).add("three times");

 //verification using never(). never() is an alias to times(0)
 verify(mockedList, never()).add("never happened");

 //verification using atLeast()/atMost()
 verify(mockedList, atLeast(2)).add("three times");
 verify(mockedList, atMost(5)).add("three times");
```

### 校验调用顺序

* InOrder类

#### 校验一个对象
```java
//create an inOrder verifier for a single mock
 InOrder inOrder = inOrder(singleMock);

 //following will make sure that add is first called with "was added first", then with "was added second"
 inOrder.verify(singleMock).add("was added first");
 inOrder.verify(singleMock).add("was added second");
```

#### 校验多个对象
```java
//create inOrder object passing any mocks that need to be verified in order
 InOrder inOrder = inOrder(firstMock, secondMock);

 //following will make sure that firstMock was called before secondMock
 inOrder.verify(firstMock).add("was called first");
 inOrder.verify(secondMock).add("was called second");
```

### 整体校验
* verifyZeroInteractions()  没有发生过任何调用
* verifyNoMoreInteractions()  除了校验过的调用，没有发生过其他调用

