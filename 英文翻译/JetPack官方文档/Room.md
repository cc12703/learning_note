

# Room

* 原始文档：https://developer.android.com/training/data-storage/room

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
* 有时候，我们会希望一个实体、数据对象在数据库逻辑中作为一个密不可分的整体处理。尽管这个对象包含了很多属性
* 这时候，我们可以使用@Embedded标注一个上面描述的对象分解成多个子类
* 你可以像查询其他独立列一样的查询这些内嵌字段

```kotlin
data class Address(
    val street: String?,
    val state: String?,
    val city: String?,
    @ColumnInfo(name = "post_code") val postCode: Int
)

@Entity
data class User(
    @PrimaryKey val id: Int,
    val firstName: String?,
    @Embedded val address: Address?
)
```
说明
* User表中包含的列：id,firstName,street,state,city,post_code


### 定义一对一关系
* 两个实体一对一关系：父实体的每个实例都精确的对应一个子实体实例，反之亦然

#### 例子
* 一个流音乐播放应用，每个用户都有一个他们自己的曲库
* 每个用户都只有一个曲库
* 每个曲库都只对应一个用户

```kotlin
@Entity
data class User(
    @PrimaryKey val userId: Long,
    val name: String,
    val age: Int
)

@Entity
data class Library(
    @PrimaryKey val libraryId: Long,
    val userOwnerId: Long
)

data class UserAndLibrary(
    @Embedded val user: User,
    @Relation(
         parentColumn = "userId",
         entityColumn = "userOwnerId"
    )
    val library: Library
)

@Transaction
@Query("SELECT * FROM User")
fun getUsersAndLibraries(): List<UserAndLibrary>
```
说明
* parentColumn 为父实体的主键名
* entityColumn 为子实体的引用父实体主键的列名
* @Transaction 为了保证操作两张表是原子的

### 定义一对多关系
* 两个实体的一对多关系：每个父实体实例对应0个、多个子实体实例。但是每个子实体实例只能对应一个父实体实例

#### 例子
* 在流媒体播放应用，每个用户都可以组织歌曲到一个播放列表中
* 每个一用户可以创建很多个播放列表，每个播放列表只能有一个用户
* User实体和Playlist实体间是一对多关系

```kotlin
@Entity
data class User(
    @PrimaryKey val userId: Long,
    val name: String,
    val age: Int
)

@Entity
data class Playlist(
    @PrimaryKey val playlistId: Long,
    val userCreatorId: Long,
    val playlistName: String
)

data class UserWithPlaylists(
    @Embedded val user: User,
    @Relation(
          parentColumn = "userId",
          entityColumn = "userCreatorId"
    )
    val playlists: List<Playlist>
)

@Transaction
@Query("SELECT * FROM User")
fun getUsersWithPlaylists(): List<UserWithPlaylists>
```
说明
**同上**

### 定义多对多关系
* 两个实体的多对多关系：每个父实体实例都对应0个、多个子节点实例，反之亦然

#### 例子
* 在流音乐播放应用中，对于用户自定义播放列表。
* 每个播放列表可以包含多个歌曲，每个歌曲可以是很多个不同播放列表的一部分
* Playlist实体和Song实体之间是多对多关系


```kotlin
@Entity
data class Playlist(
    @PrimaryKey val playlistId: Long,
    val playlistName: String
)

@Entity
data class Song(
    @PrimaryKey val songId: Long,
    val songName: String,
    val artist: String
)

//定义交叉引用表
@Entity(primaryKeys = ["playlistId", "songId"])
data class PlaylistSongCrossRef(
    val playlistId: Long,
    val songId: Long
)

data class PlaylistWithSongs(
    @Embedded val playlist: Playlist,
    @Relation(
         parentColumn = "playlistId",
         entityColumn = "songId",
         associateBy = Junction(PlaylistSongCrossRef::class)
    )
    val songs: List<Song>
)

data class SongWithPlaylists(
    @Embedded val song: Song,
    @Relation(
         parentColumn = "songId",
         entityColumn = "playlistId",
         associateBy = Junction(PlaylistSongCrossRef::class)
    )
    val playlists: List<Playlist>
)

@Transaction
@Query("SELECT * FROM Playlist")
fun getPlaylistsWithSongs(): List<PlaylistWithSongs>

@Transaction
@Query("SELECT * FROM Song")
fun getSongsWithPlaylists(): List<SongWithPlaylists>
```
说明
* 交叉引用表必须包含两个实体的主键
* associateBy用于引用交叉引用表


### 定义内嵌关系
* 用于查询一个由三张表、更多表组成的集合，表之间都有相关关系


#### 例子
* 在流音乐播放应用中，你想要查询所有用户的所有播放列表中的所有歌曲
* 用户和播放列表之间是一对多关系，播放列表和歌曲之间是多对多关系

```kotlin
@Entity
data class User(
    @PrimaryKey val userId: Long,
    val name: String,
    val age: Int
)

@Entity
data class Playlist(
    @PrimaryKey val playlistId: Long,
    val userCreatorId: Long,
    val playlistName: String
)

@Entity
data class Song(
    @PrimaryKey val songId: Long,
    val songName: String,
    val artist: String
)

@Entity(primaryKeys = ["playlistId", "songId"])
data class PlaylistSongCrossRef(
    val playlistId: Long,
    val songId: Long
)

data class PlaylistWithSongs(
    @Embedded val playlist: Playlist,
    @Relation(
         parentColumn = "playlistId",
         entityColumn = "songId",
         associateBy = @Junction(PlaylistSongCrossRef::class)
    )
    val songs: List<Song>
)

data class UserWithPlaylistsAndSongs(
    @Embedded val user: User
    @Relation(
        entity = Playlist::class,
        parentColumn = "userId",
        entityColumn = "userCreatorId"
    )
    val playlists: List<PlaylistWithSongs>
)

@Transaction
@Query("SELECT * FROM User")
fun getUsersWithPlaylistsAndSongs(): List<UserWithPlaylistsAndSongs>
```
说明
* UserWithPlaylistsAndSongs间接建模了三个实体类之间的关系：User,Playlist,Sone
* 如果你的集合有更多的表，你可以创建一个类来对所有剩余表进行建模，创建一个关系类对所有以前表的关系进行建模
* 你可以通过在所有表之间创建一个嵌套关系链来完成查询

图示
![](https://gitee.com/cc12703/figurebed/raw/master/img/20210110154654.png)





## 异步DAO查询

### 概述
* 为了防止查询会阻塞界面，Room不允许在主线程上进行数据库访问。
* 这个限制意味了你必须进行异步DAO查询

#### 查询有三种类型
* 单次写查询：插入、更新、删除数据
* 单次读查询：只从数据库中读取数据一次，返回当时时刻的数据库快照
* 可观测的读查询：当底层数据库变更时，每次都从数据库读取数据并发送变更后的新值


### 语言和框架选项
* Room支持和多种语言特性和库的整合

|查询类型|Kotlin语言特性|RxJava|Guava|JetPack生命周期|
|--|--|--|--|--|--|
|单次写|协程|Single\<T\>, Maybe\<T\>, Completable|ListenableFuture\<T\>|无|
|单次读|协程|Single\<T\>, Maybe\<T\>|ListenableFuture\<T\>|无|
|可观测读|Flow\<T\>|Flowable\<T\>, Publisher\<T\>, Observable\<T\>|无|LiveData\<T\>|

#### Kotlin
* Room 2.2版本及更高，可以使用Flow函数写可观测查询
* Room 2.1版本及更高，可以使用suspend关键字（用协程实现异步查询）

#### RxJava
* 对于单次查询，Room支持 Completable, Single\<T\>, Maybe\<T\> 三种返回类型
* 对于可观测查询，Room支持 Publisher\<T\>, Flowable\<T\>, Observable\<T\> 三种返回类型

#### Java
* 对于可观测查询，可以使用Jetpack库的LiveData包装类
* 对于单次查询，可以使用Guava库的 ListenableFuture包装类


### 异步单次查询
* 单次查询操作指查询只会执行一次，获取到执行时刻的数据快照

#### 协程
```kotlin
@Dao
interface UserDao {
    @Insert(onConflict = OnConflictStrategy.REPLACE)
    suspend fun insertUsers(vararg users: User)

    @Update
    suspend fun updateUsers(vararg users: User)

    @Delete
    suspend fun deleteUsers(vararg users: User)

    @Query("SELECT * FROM user WHERE id = :id")
    suspend fun loadUserById(id: Int): User

    @Query("SELECT * from user WHERE region IN (:regions)")
    suspend fun loadUsersByRegion(regions: List<String>): List<User>
}
```

#### RxJava
```java
@Dao
public interface UserDao {
    @Insert(onConflict = OnConflictStrategy.REPLACE)
    public Completable insertUsers(List<User> users);

    @Update
    public Completable updateUsers(List<User> users);

    @Delete
    public Completable deleteUsers(List<User> users);

    @Query("SELECT * FROM user WHERE id = :id")
    public Single<User> loadUserById(int id);

    @Query("SELECT * from user WHERE region IN (:regions)")
    public Single<List<User>> loadUsersByRegion(List<String> regions);
}
```

#### Guava
```java
@Dao
public interface UserDao {
    // Returns the number of users inserted.
    @Insert(onConflict = OnConflictStrategy.REPLACE)
    public ListenableFuture<Integer> insertUsers(List<User> users);

    // Returns the number of users updated.
    @Update
    public ListenableFuture<Integer> updateUsers(List<User> users);

    // Returns the number of users deleted.
    @Delete
    public ListenableFuture<Integer> deleteUsers(List<User> users);

    @Query("SELECT * FROM user WHERE id = :id")
    public ListenableFuture<User> loadUserById(int id);

    @Query("SELECT * from user WHERE region IN (:regions)")
    public ListenableFuture<List<User>> loadUsersByRegion(List<String> regions);
}
```


### 可观测查询
* 一种读操作，当查询引用的表发生变更时会发送新值
* 你使用这特征的一种方法是，在底层数据库被插入、修改、删除时，保持界面显示列表是最新的

#### 协程
```kotlin
@Dao
interface UserDao {
    @Query("SELECT * FROM user WHERE id = :id")
    fun loadUserById(id: Int): Flow<User>

    @Query("SELECT * from user WHERE region IN (:regions)")
    fun loadUsersByRegion(regions: List<String>): Flow<List<User>>
}
```

#### RxJava
```java
@Dao
public interface UserDao {
    @Query("SELECT * FROM user WHERE id = :id")
    public Flowable<User> loadUserById(int id);

    @Query("SELECT * from user WHERE region IN (:regions)")
    public Flowable<List<User>> loadUsersByRegion(List<String> regions);
}
```

#### JetPack
```java
@Dao
public interface UserDao {
    @Query("SELECT * FROM user WHERE id = :id")
    public LiveData<User> loadUserById(int id);

    @Query("SELECT * from user WHERE region IN (:regions)")
    public LiveData<List<User>> loadUsersByRegion(List<String> regions);
}
```

#### 注意
* Room的可观测查询有一个重要限制
    * 表中的任何行更新了，查询都会重新运行
    * 可以使用 distinctUntilChanged()来保证实际的查询结果被改变




## 视图

### 概述
* Room支持SQLite数据库视图
* 允许你将一个查询内嵌到一个类


### 创建
* 使用@DatabaseView标注一个类
* 标注值是一个查询语句

```kotlin
@DatabaseView("SELECT user.id, user.name, user.departmentId," +
        "department.name AS departmentName FROM user " +
        "INNER JOIN department ON user.departmentId = department.id")
data class UserDetail(
    val id: Long,
    val name: String?,
    val departmentId: Long,
    val departmentName: String?
)
```

### 关联视图

```kotlin
@Database(entities = arrayOf(User::class),
          views = arrayOf(UserDetail::class), version = 1)
abstract class AppDatabase : RoomDatabase() {
    abstract fun userDao(): UserDao
}
```



## 预填充数据库

### 概述
* 有时候，你希望在应用启动时，数据已经加载了一份数据。这被称为预填充数据库
* 在Room中你可以在初始化时，使用一份预打包的数据库文件来预填充数据库

### 从asset预填充
* 为了从应用asset目录下的预打包数据库文件来预填充数据库，你需要在调用build()之前调用createFromAsset()

```kotlin
Room.databaseBuilder(appContext, AppDatabase.class, "Sample.db")
    .createFromAsset("database/myapp.db")
    .build()
```

### 从文件系统预填充
* 为了从设备文件系统下的预打包数据库文件来预填充数据库，你需要在调用build()之前调用createFromFile()

```kotlin
Room.databaseBuilder(appContext, AppDatabase.class, "Sample.db")
    .createFromFile(File("mypath"))
    .build()
```


### 处理兼容性（包含预打包数据库）
#### 概述
* 预打包数据库会改变Room处理回调迁移的方式
* 原始情况，当发生破坏性迁移时，Room执行迁移时没有发现一条迁移路径，Room会丢弃所有的表，使用目标版本的数据库模式来创建一个空的数据库
* 如果你有一个和目标版本号一样的预打包数据库文件，Room会在执行破坏性迁移后使用预打包文件的内容填充新数据库

#### 例子
##### 带预打包数据库的回退迁移
###### 情况
1. 应用定义了一个版本号3的Room数据库
1. 设备上已经安装了版本号为2的数据库实例
1. 有一个版本号3的预打包数据库文件
1. 没有实现版本号2到版本号3的迁移路径
1. 破坏性迁移开启

```kotlin
// 定义版本号为3的数据库类
@Database(version = 3)
abstract class AppDatabase : RoomDatabase() {
    ...
}

// 破坏性迁移启用、提供预打包数据库
Room.databaseBuilder(appContext, AppDatabase.class, "Sample.db")
    .createFromAsset("database/myapp.db")
    .fallbackToDestructiveMigration()
    .build()
```

###### 处理流程
1. 因为应用中数据库定义为版本号3，设备上已安装的数据库实例为版本2，所有需要进行迁移
1. 因为没有定义版本2到版本3的迁移路径，所以迁移是一次回退迁移
1. 因为调用了fallbackToDestructiveMigration()，所有回退迁移变成了破坏性迁移
    * Room会丢弃已安装数据库实例中的所有表
1. 因为预打包数据库文件的版本是3，所有Room会重建数据库，并使用预打包文件的内容填充新数据库
    * 如果你的预打包文件的版本是2，Room会注意到该版本号不匹配目标版本号，将不会使用该预打包文件

##### 带迁移路径和预打包数据库
###### 情况
* 情况同上，只是实现了版本号2到版本号3的迁移路径

```kotlin
// 定义版本号为3的数据库类
@Database(version = 3)
abstract class AppDatabase : RoomDatabase() {
    ...
}

// 定义版本号2到版本号3的迁移路径
val MIGRATION_2_3 = object : Migration(2, 3) {
    override fun migrate(database: SupportSQLiteDatabase) {
        ...
    }
}

// 提供一个预打包数据库
Room.databaseBuilder(appContext, AppDatabase.class, "Sample.db")
    .createFromAsset("database/myapp.db")
    .addMigrations(MIGRATION_2_3)
    .build()
```

###### 处理流程
1. 因为应用中数据库定义为版本号3，设备上已安装的数据库实例为版本2，所有需要进行迁移
1. 因为实现了版本号2到版本号3的迁移路径
    * Room会调用migrate()来升级设备上的数据库实例为版本3，保留数据库中的数据
    * Room不会使用预打包数据库文件，因为只会在回调迁移时才会使用预打包文件

##### 带预打包数据库的多阶段迁移
###### 情况
1. 应用定义了数据库为版本4
1. 设备上已安装的数据库实例为版本2
1. 预打包数据库文件为版本3
1. 只实现了版本3到版本4的迁移路径，版本2到版本3的迁移路径没有
1. 破坏性迁移被开启

```kotlin
// 定义版本4的数据库类
@Database(version = 4)
abstract class AppDatabase : RoomDatabase() {
    ...
}

// 定义版本3到版本4的迁移路径
val MIGRATION_3_4 = object : Migration(3, 4) {
    override fun migrate(database: SupportSQLiteDatabase) {
        ...
    }
}

// 启用破坏性迁移，提供预打包数据库
Room.databaseBuilder(appContext, AppDatabase.class, "Sample.db")
    .createFromAsset("database/myapp.db")
    .addMigrations(MIGRATION_3_4)
    .fallbackToDestructiveMigration()
    .build()
```

###### 处理流程
1. 因为应用中数据库定义为版本号4，设备上已安装的数据库实例为版本2，所有需要进行迁移
1. 因为没有定义版本2到版本3的迁移路径，所以迁移是一次回退迁移
1. 因为调用了fallbackToDestructiveMigration()，所有回退迁移变成了破坏性迁移
    * Room会丢弃已安装数据库实例中的所有表
1. 因为提供了版本3的预打包数据库文件，Room会重新数据库，并从预打包文件的数据填充数据库
1. 目前为止，设备上已安装的数据库版本已经变成了3
1. 因为该版本号依然低于应用中定义的版本号，还有进行一次迁移
1. 因为定义了版本3到版本4的迁移路径
    * Room会调用migrate()来升级设备中的数据库实例为版本4




## 迁移数据库

### 概述
* 随着应用的升级，你需要不停修改实体类已反应这些变化
* 但是你需要在升级应用修改数据库模式时，保留用户已经的产生的数据


### 迁移
* Room通过Migrate类支持增量式迁移
* 每个Migrate子类，通过重写migrate方法，在startVersion和endVersion之间定义了一个迁移路径
* 当应用需要升级数据库时，Room会运行多个Migrate子类的migrate函数，将数据库迁移到最新版本
* 当迁移操作完成后，Room会校验数据库模式，确保迁移成功。如果Room发现问题，会抛出异常信息

#### 例子
```kotlin
val MIGRATION_1_2 = object : Migration(1, 2) {
    override fun migrate(database: SupportSQLiteDatabase) {
        database.execSQL("CREATE TABLE `Fruit` (`id` INTEGER, `name` TEXT, " +
                "PRIMARY KEY(`id`))")
    }
}

val MIGRATION_2_3 = object : Migration(2, 3) {
    override fun migrate(database: SupportSQLiteDatabase) {
        database.execSQL("ALTER TABLE Book ADD COLUMN pub_year INTEGER")
    }
}

Room.databaseBuilder(applicationContext, MyDb::class.java, "database-name")
        .addMigrations(MIGRATION_1_2, MIGRATION_2_3).build()
```


### 测试迁移
#### 概述
* 迁移一般都比较复杂，不正确定义的迁移会导致应用的崩溃。为了应用的稳定性，你需要测试你的迁移
* Room提供了一个room-test的插件用于辅助测试流程

#### 导出数据库模式
* Room会在编译阶段，将数据库模式导出为JSON文件
* 为了导出数据库模式，你需要在build.gradle中设置room.schemaLocation标注处理器
* 导出的JSON文件代表了你数据库模式的历史，你需要将其保存到版本控制系统中
* 这会允许Room为了测试目的创建旧版本的数据库

```groovy
android {
    ...
    defaultConfig {
        ...
        javaCompileOptions {
            annotationProcessorOptions {
                arguments += ["room.schemaLocation":
                             "$projectDir/schemas".toString()]
            }
        }
    }
}
```


#### 添加依赖
* 在测试迁移前，需要增加 androidx.room:room-testing库
* 添加存放导出的数据库模式文件的目录作为asset目录

```groovy
android {
    ...
    sourceSets {
        // 添加导出的模式文件目录作为测试应用的asset
        androidTest.assets.srcDirs += files("$projectDir/schemas".toString())
    }
}

dependencies {
    ...
      testImplementation "androidx.room:room-testing:2.2.6"
}
```

#### 测试单一迁移
* 测试库提供MigrationTestHelper类，来读取导出的模式文件
* 该类还实现了JUnit4的TestRule接口，可以管理已创建的数据库

```kotlin
@RunWith(AndroidJUnit4::class)
class MigrationTest {
    private val TEST_DB = "migration-test"

    @Rule
    val helper: MigrationTestHelper = MigrationTestHelper(
            InstrumentationRegistry.getInstrumentation(),
            MigrationDb::class.java.canonicalName,
            FrameworkSQLiteOpenHelperFactory()
    )

    @Test
    @Throws(IOException::class)
    fun migrate1To2() {
        var db = helper.createDatabase(TEST_DB, 1).apply {
            // 数据库模式为版本1，使用SQL语句插入一些数据
            // 你不能使用DAO类，应该那个使用了最新的模式
            execSQL(...)

            // 为下个版本做准备
            close()
        }

        // 使用版本2重新打开数据，并提供MIGRATION_1_2作为迁移流程
        db = helper.runMigrationsAndValidate(TEST_DB, 2, true, MIGRATION_1_2)

        // MigrationTestHelper 会自动校验数据库模式的变更
        // 你需要自己校验数据是否被正确迁移
    }
}
```

#### 测试全部迁移
* 推荐测试时覆盖数据库的所有迁移过程
* 这样做可以确保最近创建的数据库实例之间没有矛盾，一个旧实例可以沿着已定义的迁移流程进行迁移

```kotlin
@RunWith(AndroidJUnit4::class)
class MigrationTest {
    private val TEST_DB = "migration-test"

    // 定义所有的迁移
    private val ALL_MIGRATIONS = arrayOf(
            MIGRATION_1_2, MIGRATION_2_3, MIGRATION_3_4)

    @Rule
    val helper: MigrationTestHelper = MigrationTestHelper(
            InstrumentationRegistry.getInstrumentation(),
            AppDatabase::class.java.canonicalName,
            FrameworkSQLiteOpenHelperFactory()
    )

    @Test
    @Throws(IOException::class)
    fun migrateAll() {
        // 创建数据库的最早版本
        helper.createDatabase(TEST_DB, 1).apply {
            close()
        }

        // 打开数据库最新版本。Room会在所有迁移完成后，校验数据库模式
        Room.databaseBuilder(
                InstrumentationRegistry.getInstrumentation().getTargetContext(),
                AppDatabase.class,
                TEST_DB
        ).addMigrations(*ALL_MIGRATIONS).build().apply {
            getOpenHelper().getWritableDatabase()
            close()
        }
    }
}
```


### 优雅处理丢失的迁移路径
#### 概述
* 如果Room在升级数据库到当前版本时，没有找到一个迁移路径，会抛出一个IllegalStateException异常
* 如果你可以接受在无法迁移时丢失数据，你可以调用fallbackToDestructiveMigration()
* 这个方法告诉Room如果在执行增量式迁移时，没有找到迁移路径，则可以进行破坏性迁移

```kotlin
Room.databaseBuilder(applicationContext, MyDb::class.java, "database-name")
        .fallbackToDestructiveMigration()
        .build()
```

#### 特定情况进行破坏性迁移
* 如果在特定版本上出错了，而你有无法使用迁移流程解决这个错误，你可以使用fallbackToDestructiveMigrationFrom()
    * 告诉Room只有从特点版本上迁移时才进入破坏性迁移
* 如果想在从高版本向低版本迁移时才使用破坏性迁移，你可以使用fallbackToDestructiveMigrationOnDowngrade()



### 升级到Room2.2.0时处理列默认值
#### 概述
* 在Room2.2.0以上，你可以使用标注 @ColumnInfo(defaultValue = "...")来定义
* 在Room2.2.0以下，定义默认值的唯一方法就是直接执行SQL语句，这意味着创建的默认值Room是不知道的

#### 问题
```kotlin
//使用Room 2.1.0, Song实体，数据库版本号1
@Entity
data class Song(
    @PrimaryKey
    val id: Long,
    val title: String
)

//使用Room 2.1.0, Song实体，数据库版本号2
@Entity
data class Song(
    @PrimaryKey
    val id: Long,
    val title: String,
    val tag: String // 在版本2中增加
)

// 从版本1迁移到版本2
val MIGRATION_1_2 = object : Migration(1, 2) {
    override fun migrate(database: SupportSQLiteDatabase) {
        database.execSQL(
            "ALTER TABLE Song ADD COLUMN tag TEXT NOT NULL DEFAULT ''")
    }
}
```
说明
* 该应用的数据库在升级和重新安装时会产生不兼容
    * 因为tag的默认值只会在从版本1到版本2的迁移操作中被定义
    * 任何从版本2开始安装应用的用户，tag都不会有默认值
* 在Room2.2.0以下，这个差异是无害的
* 然而如果应用升级到Room 2.2.0或以上，并且修改了Song类，使用@ColumnInfo给tag设置了默认值。这个时候Room就会发现这个差异，并导致模式校验失败

#### 解决方案
为了保证数据库模式在所有用户上都是连续一致的，需要进行以下步骤

1. 使用@ColumnInfo定义各自实体类中的列的默认值
1. 升级数据库版本号，加一
1. 使用丢弃和重建策略，定义一个迁移路径，给已经存在的列增加一个默认值

```kotlin
// Room2.2.0，版本2迁移到版本3
val MIGRATION_2_3 = object : Migration(2, 3) {
    override fun migrate(database: SupportSQLiteDatabase) {
        database.execSQL("""
                CREATE TABLE new_Song (
                    id INTEGER PRIMARY KEY NOT NULL,
                    name TEXT,
                    tag TEXT NOT NULL DEFAULT ''
                )
                """.trimIndent())
        database.execSQL("""
                INSERT INTO new_Song (id, name, tag)
                SELECT id, name, tag FROM Song
                """.trimIndent())
        database.execSQL("DROP TABLE Song")
        database.execSQL("ALTER TABLE new_Song RENAME TO Song")
    }
}
```


## 测试和调试

### 测试数据库
#### 在android设备上测试
* 推荐的测试数据库的方法：写JUnit测试用例，在android设备上运行
    * 这些测试不需要创建activity，可以比界面测试更快速的执行
* 当设置好测试用例，你可以创建一个内存版本的数据库，可以使你的测试更封闭

```kotlin
@RunWith(AndroidJUnit4::class)
class SimpleEntityReadWriteTest {
    private lateinit var userDao: UserDao
    private lateinit var db: TestDatabase

    @Before
    fun createDb() {
        val context = ApplicationProvider.getApplicationContext<Context>()
        db = Room.inMemoryDatabaseBuilder(
                context, TestDatabase::class.java).build()
        userDao = db.getUserDao()
    }

    @After
    @Throws(IOException::class)
    fun closeDb() {
        db.close()
    }

    @Test
    @Throws(Exception::class)
    fun writeUserAndReadInList() {
        val user: User = TestUtil.createUser(3).apply {
            setName("george")
        }
        userDao.insert(user)
        val byName = userDao.findUsersByName("george")
        assertThat(byName.get(0), equalTo(user))
    }
}
```

#### 在开发机上测试
* Room使用SQLte支持库，提供一个接口用于匹配android框架
* 这个特性可以允许你传入一个自定义的实现来测试你的数据库


## 调试数据库

### 使用数据库检测器
* Android Stuido 4.1及其以上，数据库检测器允许你在应用运行时，检查、查询、修改你的数据库
* 这个数据库检测器兼容android绑定的多个版本的SQLite
* 带有以下几个特点
    * 使用槽动作(gutter actions)快速通过DAO类查询数据
    * 当应用变更数据时，可以立刻在检测器上看到变化

### 从命令行导出数据
* Android SDK包含一组sqlite3的数据库工具，用于检查应用的数据库
* 其中 .dump 用于打印表中的数据
* .schema 用于打印已存在表的SQL CREATE语句

#### 执行命令
```shell
adb -s emulator-5554 shell
sqlite3 /data/data/your-app-package/databases/rssitems.db 
```