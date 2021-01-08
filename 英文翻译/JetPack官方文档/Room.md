

# Room

*原始文档：https://developer.android.com/training/data-storage/room

[TOC]

## 概述

### 总体
#### 持久化数据
* 应用在处理有意义的结构化数据试，将其保存在本地可以获得巨大的好处
    * 例子：缓存相关数据，当设备无法联网时，可以使用缓存数据继续展示

#### 作用
* Room提供了一个基于SQLite的抽象层
* 允许使用流式接口访问数据库，支持SQLite的所有功能

#### 额外优点
* 提供对SQL查询语句的编译时校验
* 方便的标注，最小化重复的、容易出错的模板代码
* 流式的数据库迁移路径

### 安装
* 在build.gradle中添加依赖

```kotlin

```
dependencies {
  def room_version = "2.2.6"

  implementation "androidx.room:room-runtime:$room_version"
  kapt "androidx.room:room-compiler:$room_version"

  // 可选的 - Kotlin扩展和协程支持
  implementation "androidx.room:room-ktx:$room_version"

  // 可选的 - 测试辅助
  testImplementation "androidx.room:room-testing:$room_version"
}
```

### 主要组件
* 数据库类：作为主访问点，用于与应用持久化数据的底层连接
* 数据实体：代表数据库中的数据表
* DAO：提供方法来操作数据库（查询、更新、插入、删除数据）

#### 组件关系
* 数据库类给应用提供了一个数据库的实例，DAO会关联在上面
* 应用可以使用DAO来检索数据从对应的数据实例上
* 应用也可以使用一个已定义好的数据实例来更新对应表的数据行、创建新的数据行

#### 架构图
![](https://gitee.com/cc12703/figurebed/raw/master/img/20210105110534.png)


### 简单例子
#### 数据实例
```kotlin
@Entity
data class User(
    @PrimaryKey val uid: Int,
    @ColumnInfo(name = "first_name") val firstName: String?,
    @ColumnInfo(name = "last_name") val lastName: String?
)
```

#### DAO
```kotlin
@Dao
interface UserDao {
    @Query("SELECT * FROM user")
    fun getAll(): List<User>

    @Query("SELECT * FROM user WHERE uid IN (:userIds)")
    fun loadAllByIds(userIds: IntArray): List<User>

    @Query("SELECT * FROM user WHERE first_name LIKE :first AND " +
           "last_name LIKE :last LIMIT 1")
    fun findByName(first: String, last: String): User

    @Insert
    fun insertAll(vararg users: User)

    @Delete
    fun delete(user: User)
}
```

#### 数据库
```kotlin
@Database(entities = arrayOf(User::class), version = 1)
abstract class AppDatabase : RoomDatabase() {
    abstract fun userDao(): UserDao
}
```
##### 要点
* 类必须使用@Database进行标注，包含所有的数据类
* 类必须是抽象类，继承于RoomDatabase
* 每个DAO对应一个抽象方法


#### 使用
```kotlin
val db = Room.databaseBuilder(
            applicationContext,
            AppDatabase::class.java, "database-name"
        ).build()

val userDao = db.userDao()
val users: List<User> = userDao.getAll()
```


## 定义数据实体

### 概述
* 使用room时，你可以用实体定义一组相关的属性
* 每一个实体，会对应创建一张表，保存所有的数据项
* 你需要在Database类中引用所有用到的实体类

#### 例子
```kotlin
@Entity
data class User(
    @PrimaryKey var id: Int,
    var firstName: String?,
    var lastName: String?
)
```
说明
* 为了持久化属性，Room需要能够访问这些属性
* 你可以将属性设置为公共、提供getter, setter函数
* 如果要使用getter,setter函数，需要记住，它们要遵守JavaBeans协议


### 主键
* 每个实体必须要指定一个属性为主键
* 如果只有一个主键，可以使用@PrimaryKey标注
* 如果有一个复合主键，则需要在@Entity中配置primaryKeys属性
* 如果需要使用自增加ID，则需要设置@PrimaryKey的autoGenerate属性

```kotlin
@Entity(primaryKeys = arrayOf("firstName", "lastName"))
data class User(
    val firstName: String?,
    val lastName: String?
)
```

### 名字
* 默认情况下，Room使用类名作为数据库表的名字
* 可以用@Entity的tableName指定名字
* 类似的，Room使用属性名字作为数据库表的列名
* 可以用@ColumnInfo来指定名字

```kotlin
@Entity(tableName = "users")
data class User (
    @PrimaryKey val id: Int,
    @ColumnInfo(name = "first_name") val firstName: String?,
    @ColumnInfo(name = "last_name") val lastName: String?
)
```

### 忽略属性
* 默认，Room使用实体类的每个属性来创建表的列
* 可以使用@Ignore来忽略属性
* 也可以使用@Entity的ignoredColumns属性

```kotlin
@Entity
data class User(
    @PrimaryKey val id: Int,
    val firstName: String?,
    val lastName: String?,
    @Ignore val picture: Bitmap?
)
```

```kotlin
open class User {
    var picture: Bitmap? = null
}

@Entity(ignoredColumns = arrayOf("picture"))
data class RemoteUser(
    @PrimaryKey val id: Int,
    val hasVpn: Boolean
) : User()
```

### 表搜索支持
* Room提供了很多标注，来使搜索数据库表的内容更容易

#### 全文搜索
* 如果应用需要通过FTS(全文搜索)来加快访问数据库信息时，需要实体有使用了FTS3,FTS4功能的虚拟表来支持
* Room提供了很多选项用于配置FTS功能的实体，可以通过FtsOptions来设置

```kotlin
// 如果你的应用有严格的磁盘空间要求、或需要兼容老版本的SQLite，则可以使用@Fts3。
// 不然建议使用@Fts4
// 通过使用languageId来使表支持多语言
@Fts4(languageId = "lid")
@Entity(tableName = "users")
data class User(
    //指定主键不是必须的，但是若要指定，则必要定义以下格式
    @PrimaryKey @ColumnInfo(name = "rowid") val id: Int,
    @ColumnInfo(name = "first_name") val firstName: String?
)
```

#### 列索引
* 如果你应用使用的SDK版本不允许使用FTS功能的实体，你仍然可以通过索引指定列来加快查询操作
* 通过@Entity中的indices属性可以配置需要索引的列名
* 可以通过设置unique为true，可以保证属性、属性组的值是唯一的

```kotlin
@Entity(indices = arrayOf(Index(value = ["last_name", "address"])))
data class User(
    @PrimaryKey val id: Int,
    val firstName: String?,
    val address: String?,
    @ColumnInfo(name = "last_name") val lastName: String?,
    @Ignore val picture: Bitmap?
)
```

```kotlin
@Entity(indices = arrayOf(Index(value = ["first_name", "last_name"],
        unique = true)))
data class User(
    @PrimaryKey val id: Int,
    @ColumnInfo(name = "first_name") val firstName: String?,
    @ColumnInfo(name = "last_name") val lastName: String?,
    @Ignore var picture: Bitmap?
)
```

### 使用AutoValue对象
* 该特性只为基于Java实体而设计，为了达到基于Kotlin实体的功能



## 用DAO存取数据

### 概述
* 使用Room来保存数据后，你需要通过DAO来与保存的数据交互
* 每个DAO包含一些方法，可以提供对数据库的抽象访问能力
* 在编译时，Room会自动生成你定义的DAO类的实现类

#### 优点
* 通过使用DAO，可以代替使用查询生成器、直接查询。可以让你保留关注点分离
* 通过DAO可以更容易的使用mock数据库来测试应用


### DAO剖析
* 可以使用接口类、抽象类来定义DAO。一般情况下会使用接口类
* 需要使用@Dao来标注类
* DAO没有属性，通过定义方法来与数据进行交互

#### 方法分类
* 快捷方法：用于插入、更新、删除数据行，而不用写SQL语句
* 查询方法：需要写SQL语句

```kotlin
@Dao
interface UserDao {
    @Insert
    fun insertAll(vararg users: User)

    @Delete
    fun delete(user: User)

    @Query("SELECT * FROM user")
    fun getAll(): List<User>
}
```


### 快捷方法
* Room提供了快捷方法来在不需要写SQL语句时，进行插入、更新、删除操作
* 如果需要复杂的插入、更新、删除操作，就需要使用查询方法来实现

#### 插入
* @Insert标注允许你定义方法，来将参数插入到对应的数据库表中
* 方法参数必须是Room数据实体的实例、是包含了实体实例的集合
* 当方法参数是一个对象时，会返回一个long值，是插入数据项的rowID
* 当方法参数是一个集合、列表时，会返回一个long值的集合，内容是rowID

```kotlin
@Dao
interface UserDao {
    @Insert(onConflict = OnConflictStrategy.REPLACE)
    fun insertUsers(vararg users: User)

    @Insert
    fun insertBothUsers(user1: User, user2: User)

    @Insert
    fun insertUsersAndFriends(user: User, friends: List<User>)
}
```

#### 更新
* @Update标注允许你定义方法，来更新数据库表中的指定行
* 方法参数必须是数据实体的实例
* Room使用主键去查找传入实体在数据库中所在的行。如果查找不到，则不会更新
* 方法可以选择返回一个int值，表明更新成功的行数

```kotlin
@Dao
interface UserDao {
    @Update
    fun updateUsers(vararg users: User)
}
```

#### 删除
* @Delete标注允许你定义方法，来删除数据库表中的指定行
* 方法参数必须是数据实体的实例
* Room使用主键去查找传入实体在数据库中所在的行。如果查找不到，则不会更新
* 方法可以选择返回一个int值，表明更新成功的行

```kotlin
@Dao
interface UserDao {
    @Delete
    fun deleteUsers(vararg users: User)
}
```



### 查询方法
#### 概述
* @Query标注允许你写SQL语句并将其导出为DAO方法
* 使用查询方法可以用于从数据库中查询数据、执行复杂的插入、更新、删除操作
* Room在编译阶段验证SQL查询语句

#### 简单查询
```kotlin
//返回所有的User对象
@Query("SELECT * FROM user")
fun loadAllUsers(): Array<User>
```

#### 返回部分列的子集合
* 大部分时候，我们只想查询一部分数据表中的列
* Room允许你从任何查询中返回一个简单对象。只要你能将结果列映射到对象中

```kotlin
//Room会将first_name, last_name列的值，映射到NameTuple的对应字段上
data class NameTuple(
    @ColumnInfo(name = "first_name") val firstName: String?,
    @ColumnInfo(name = "last_name") val lastName: String?
)

@Query("SELECT first_name, last_name FROM user")
fun loadFullName(): List<NameTuple>
```

#### 传入简单参数
* 大部分时候，DAO方法需要接收参数用于执行过滤操作
* Room支持将方法参数绑定到查询语句中

```kotlin
//参数minAge会替换掉语句中的 :minAge
@Query("SELECT * FROM user WHERE age > :minAge")
fun loadAllUsersOlderThan(minAge: Int): Array<User>

@Query("SELECT * FROM user WHERE age BETWEEN :minAge AND :maxAge")
fun loadAllUsersBetweenAges(minAge: Int, maxAge: Int): Array<User>

@Query("SELECT * FROM user WHERE first_name LIKE :search " +
       "OR last_name LIKE :search")
fun findUserWithName(search: String): List<User>
```

#### 传入集合参数
* 有时候，DAO方法需要你传入一个可变数量的参数，具体值要到运行时才知道
* 当参数是一个集合时，Room在运行时，会根据提供的参数数量自动展开它们

```kotlin
@Query("SELECT * FROM user WHERE region IN (:regions)")
fun loadUsersFromRegions(regions: List<String>): List<User>
```

#### 查询多张表
* 有时候，你需要读取多张表来计算结果
* 你可以在SQL查询语句中使用JOIN子句来引用多张表
* 你也可以定义一个简单对象来返回合并表中的部分列


```kotlin
@Query(
    "SELECT * FROM book " +
    "INNER JOIN loan ON loan.book_id = book.id " +
    "INNER JOIN user ON user.id = loan.user_id " +
    "WHERE user.name LIKE :userName"
)
fun findBooksBorrowedByNameSync(userName: String): List<Book>



interface UserBookDao {
    @Query(
        "SELECT user.name AS userName, book.name AS bookName " +
        "FROM user, book " +
        "WHERE user.id = book.user_id"
    )
    fun loadUserAndBookNames(): LiveData<List<UserBook>>

    data class UserBook(val userName: String?, val bookName: String?)
}
```


### 指定返回类型
#### 带分页的查询
* Room通过整合Paging库，可以支持带分页的查询
* DAO会返回PagingSource对象

```kotlin
@Dao
interface UserDao {
  @Query("SELECT * FROM users WHERE label LIKE :query")
  fun pagingSource(query: String): PagingSource<Int, User>
}
```

#### 直接游标访问
* 如果应用逻辑要求直接访问并返回行，可以让DAO方法返回一个Cursor对象

```kotlin
@Dao
interface UserDao {
    @Query("SELECT * FROM user WHERE age > :minAge LIMIT 5")
    fun loadRawUsersOlderThan(minAge: Int): Cursor
}
```




## 定义对象间关系

### 概述
* 因为SQLite是一个关系性数据库，你可以指定数据实体之间的关系
* 尽管大部分ORM库都运行实体对象引用其他实体对象，但是Room明确禁止这么做


### 创建内嵌对象

