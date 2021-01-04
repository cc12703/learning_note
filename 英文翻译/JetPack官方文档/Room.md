

# Room

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