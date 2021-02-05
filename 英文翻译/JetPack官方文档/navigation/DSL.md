

# DSL

* 原始文档：https://developer.android.com/guide/navigation/navigation-kotlin-dsl


## 概述
* 导航组件提供了基于Kotlin的DSL，依赖于Kotlin的类型安全生成器
* 这些API允许你使用Kotlin代码来定义导航图，而不是XML
* 当你要动态生成导航时，这会非常有用
    * 例子：从服务器上下载、缓存导航配置，在activity的onCreate()中动态生成导航图



## 依赖
在应用build.gradle中增加依赖
```groovy
dependencies {
    def nav_version = "2.3.3"

    api "androidx.navigation:navigation-fragment-ktx:$nav_version"
}
```



## 生成导航图

### 例子
* 该例子来源于Sunflower应用，包含两个目的地：**家** 和 **植物详情**
* **家** 目的地是用户首次启动应用时呈现的，已列表形式显示了用户花园中的植物
* 当用户选中其中一个植物时，应用会导航到**植物详情**目的地

#### 图示
![](https://gitee.com/cc12703/figurebed/raw/master/img/20210212221952.png)


### 创建宿主
* 无论如何构建导航图，都需要在NavHost中建立导航图
* Sunflower使用fragment，所以需要NavHostFragment来代替FragmentContainerView
* 示例
    ```xml
    <!-- activity_garden.xml -->
    <FrameLayout
        xmlns:android="http://schemas.android.com/apk/res/android"
        xmlns:app="http://schemas.android.com/apk/res-auto">
        android:layout_width="match_parent"
        android:layout_height="match_parent">

        <androidx.fragment.app.FragmentContainerView
            android:id="@+id/nav_host"
            android:name="androidx.navigation.fragment.NavHostFragment"
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            app:defaultNavHost="true" />

    </FrameLayout>
    ```

### 生成常量
#### 基于XML
* 当使用基于XML的导航图时，android构建程序会解析图定义文件，会为导航图中的每个ID属性定义数字常量
* 这些常量可以在代码中通过R.id来存取
* 示例
    ```xml
    <navigation ...>
    <fragment android:id="@+id/home" ... />
    ...
    </navigation>
    ```
    说明：构建系统会生成一个常量值，R.id.home

#### 手动定义
* 在使用DSL时，解析和生成常量的步骤是不会执行的
* 你必须为每一个目的地、动作、带id值的参数自定义常量
* id值必须是唯一的和一致的
* 一种组织方式是创建一个Kotlin对象的嵌套集合
* 定义示例
    ```kotlin
    object nav_graph {

        const val id = 1 // graph id

        object dest {
            const val home = 2
            const val plant_detail = 3
        }

        object action {
            const val to_plant_detail = 4
        }

        object args {
            const val plant_id = "plantId"
        }
    }
    ```
* 使用示例
    ```kotlin
    nav_graph.id                     // 导航图ID
    nav_graph.dest.home              // 家目的地ID
    nav_graph.action.to_plant_detail // 动作 家 -> 植物详情 ID
    nav_graph.args.plant_id          // 目的地参数名字
    ```

### 构建导航图
* 使用NavController.createGraph()来创建NavGraph
    * 传入导航图ID，起始目的地ID，尾lambda用于定义导航图结构
* 可以在activity的onCreate()中构建导航图
* createGraph()返回一个Navgraph对象，可以赋值给NavController的graph属性

#### 示例
```kotlin
class GardenActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_garden)

        val navHostFragment = supportFragmentManager
                .findFragmentById(R.id.nav_host) as NavHostFragment

        navHostFragment.navController.apply {
            graph = createGraph(nav_graph.id, nav_graph.dest.home) {
                fragment<HomeViewPagerFragment>(nav_graph.dest.home) {
                    label = getString(R.string.home_title)
                    action(nav_graph.action.to_plant_detail) {
                        destinationId = nav_graph.dest.plant_detail
                    }
                }
                fragment<PlantDetailFragment>(nav_graph.dest.plant_detail) {
                    label = getString(R.string.plant_detail_title)
                    argument(nav_graph.args.plant_id) {
                        type = NavType.StringType
                    }
                }
            }
        }
    }
}
```
说明
* 使用了fragment()生成函数，来定义了两个fragment目的地
    * 需要传入目的地ID，一个可选的lambda用于附加配置（标签、动作、参数、深链）
    * Fragment类也可以通过<>来传入（类似于XML方式时，在fragment目的地设置android:name属性）


### 导航
* 当构建了导航图，就可以使用NavController.navigate()来导航了

#### 示例
```kotlin
private fun navigateToPlant(plantId: String) {

    val args = bundleOf(nav_graph.args.plant_id to plantId)

    findNavController().navigate(nav_graph.action.to_plant_detail, args)
}
```


## 支持的目的地类型

### fragment目的地
* fragment()可以通过Fragment类来参数化
* 该函数要传入一个目的地的唯一ID，以及一个lambda提供附加配置
* 示例
    ```kotlin
    fragment<FragmentDestination>(nav_graph.dest.fragment_dest_id) {
    label = getString(R.string.fragment_title)
    // arguments, actions, deepLinks...
    }
    ```

### activity目的地
* activity()不能进行参数化，可以在lambda中设置一个可选的activityClass
    * 这种灵活性允许一个activity目的地，该activity从一个隐式intent中被启动的
* 该函数要传入一个目的地的唯一ID，以及一个lambda提供附加配置
* 示例
    ```kotlin
    activity(nav_graph.dest.activity_dest_id) {
        label = getString(R.string.activity_title)
        // arguments, actions, deepLinks...

        activityClass = ActivityDestination::class
    }
    ```

### 导航图目的地
* 使用navigation()来构建一个内嵌的导航图
* 和其他目的地类型一样，该函数需要三个参数
    * 导航图的唯一ID
    * 该图起始目的地的唯一ID
    * 一个lambda用于配置该导航图
* 示例
    ```kotlin
    navigation(nav_graph.dest.nav_graph_dest, nav_graph.dest.start_dest) {
    // label, arguments, actions, other destinations, deep links
    }
    ```

### 自定义目的地
* 使用 addDestination()来添加一个自定义类型
    ```kotlin
    // NavigatorProvider 可以从 NavController 中获取
    val customDestination = navigatorProvider[CustomNavigator::class].createDestination().apply {
        id = nav_graph.dest.custom_dest_id
    }
    addDestination(customDestination)
    ```
* 使用 + 操作符来添加一个自定义类型
    ```kotlin
    // NavigatorProvider 可以从 NavController 中获取
    +navigatorProvider[CustomNavigator::class].createDestination().apply {
        id = nav_graph.dest.custom_dest_id
    }
    ```


    ### 目的地参数
    * 可以给每个目的地类型定义一个可选、必须的参数
    * 可以通过NavDestinationBuilder的argument()函数来定义一个参数
    * 需要两个参数
        * 目的地参数的名字，字符串形式
        * lambda用于构造、配置NavArgument
    * 示例
        ```kotlin
        fragment<PlantDetailFragment>(nav_graph.dest.plant_detail) {
            label = getString(R.string.plant_details_title)
            argument(nav_graph.args.plant_name) {
                type = NavType.StringType
                defaultValue = getString(R.string.default_plant_name)
                nullable = true  // default false
            }
        }
        ```



## 动作
* 可以在任何目的地（包括全局动作所在的根导航图）中定义动作
* 使用 NavDestinationBuilder.action() 来定义
* 需要传入动作的唯一ID和用于提供额外配置的lambda
* 示例
    ```kotlin
    action(nav_graph.action.to_plant_detail) {
        destinationId = nav_graph.dest.plant_detail
        navOptions {
            anim {
                enter = R.anim.nav_default_enter_anim
                exit = R.anim.nav_default_exit_anim
                popEnter = R.anim.nav_default_pop_enter_anim
                popExit = R.anim.nav_default_pop_exit_anim
            }
            popUpTo(nav_graph.dest.start_dest) {
                inclusive = true // default false
            }
            // if popping exclusively, you can specify popUpTo as
            // a property. e.g. popUpTo = nav_graph.dest.start_dest
            launchSingleTop = true // default false
        }
    }
    ```



## 深链

### 显式深链
* 可以在任何目的地中增加显式深链，就像基于XML的导航图一样

### 隐式深链



## 创建唯一ID
* 导航库要求导航图中的每个元素都要有一个ID值，并在在配置变更后保持一致性
* 一个创建方式就是将它们定义成为静态常量
* 另一种方法就是动态构造ID
    
### 示例
```kotlin
object nav_graph {
    // Counter for id's. First ID will be 1.
    var id_counter = 1

    val id = id_counter++

    object dest {
       val home = id_counter++
       val plant_detail = id_counter++
    }

    object action {
       val to_plant_detail = id_counter++
    }

    object args {
       const val plant_id = "plantId"
    }
}
```