

# ViewBinding

* 原始文档：https://developer.android.com/topic/libraries/view-binding


## 概述
* view-binding允许你更容易的写与view交互的代码
* 一旦模块中该功能被启用，每个xml布局文件都会生成一个对应的binding类
* binding类的实例包含了布局中有ID定义的view的引用
* 大部分情况下，view-binding可以替换findViewById函数

## 设置
* 在build.gradle中配置viewBinding，用于使能view-binding功能
* 在布局文件中配置viewBindingIgnore，用于禁止生成binding类

```gradle
android {
    ...
    buildFeatures {
        viewBinding true
    }
}
```

```xml
<LinearLayout
        ...
        tools:viewBindingIgnore="true" >
    ...
</LinearLayout>
```

## 使用
### 组成
* 每个布局文件生成一个对应binding类
* 每个binding类包含根view的引用，所有定义ID了的view的引用
* binding类名又xml文件名转换过来：首字母大写，尾部加Binding

#### 例子
##### 布局文件
```xml
<!-- result_profile.xml -->
<LinearLayout ... >
    <TextView android:id="@+id/name" />
    <ImageView android:cropToPadding="true" />
    <Button android:id="@+id/button"
        android:background="@drawable/rounded_button" />
</LinearLayout>
```

##### binding类
* 类名：ResultProfileBinding
* 有两个属性：nam, button
* getRoot()方法，返回布局文件根view的引用


### activity中使用binding类
* 在onCreate()中执行以下步骤
    1. 调用binding类的inflate()，生成binding实例
    1. 调用getRoot()，获取根view
    1. 调用setContentView()设置根view，在屏幕上显示

#### 例子
```kotlin
private lateinit var binding: ResultProfileBinding

override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)
    binding = ResultProfileBinding.inflate(layoutInflater)
    val view = binding.root
    setContentView(view)
}

binding.name.text = viewModel.name
binding.button.setOnClickListener { viewModel.userClicked() }
```

### fragment中使用binding类
* 在onCreateView()中执行以下步骤
    1. 调用binding类的inflate()，生成binding实例
    1. 调用getRoot()，获取根view
    1. 将根view作为返回值

#### 例子
```kotlin
private var _binding: ResultProfileBinding? = null
private val binding get() = _binding!!

override fun onCreateView(
    inflater: LayoutInflater,
    container: ViewGroup?,
    savedInstanceState: Bundle?
): View? {
    _binding = ResultProfileBinding.inflate(inflater, container, false)
    val view = binding.root
    return view
}

override fun onDestroyView() {
    super.onDestroyView()
    _binding = null
}

binding.name.text = viewModel.name
binding.button.setOnClickListener { viewModel.userClicked() }
```

## 优势

### findViewById相比
view-binding与findViewById相比有以下几个优点
* 空指针安全：binding类中的引用没有应无效ID导致的空指针的风险
* 类型安全：binding类中的引用都有标记相对应view的数据类型


### data-binding相比
veiw-binding着眼于简单的使用场景，有以下几个优点
* 更快的编译：不需要枚举处理，可以编译更快
* 方便使用：不需要特殊标记布局文件，可以更快的在项目中被采用

### 建议
* 在项目中同时使用view-binding和data-binding