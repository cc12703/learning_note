
# Room学习


## 使用

### 标注

* **@Database**: 用于创建数据库，使用在类上，该类必须继承RoomDatabase
* **@Entity**: 用于创建数据库表，使用在实体类上
* **@Dao**: 用于生成操作数据的方法，使用在类上

### 例子

```java

@Entity
public class User {
    @PrimaryKey
    private int uid;
 
    @ColumnInfo(name = "first_name")
    public String firstName;
 
    @ColumnInfo(name = "last_name")
    public String lastName;
}


@Dao
public interface UserDao {
    @Query("SELECT * FROM user")
    List<User> getAll();
 
    @Query("SELECT * FROM user WHERE uid IN (:userIds)")
    List<User> loadAllByIds(int[] userIds);
 
    @Query("SELECT * FROM user WHERE first_name LIKE :first AND "
           + "last_name LIKE :last LIMIT 1")
    User findByName(String first, String last);
 
    @Insert
    void insertAll(User... users);
 
    @Delete
    void delete(User user);
}


@Database(entities = {User.class}, version = 1)
public abstract class AppDatabase extends RoomDatabase {
    public abstract UserDao userDao();
}


//获取数据库实例
AppDatabase db = Room.databaseBuilder(getApplicationContext(),
        AppDatabase.class, "database-name").build();

```


### Entity

* 默认类中的每个字段都会创建一个列
* Ignore 忽略指定字段
* PrimaryKey  指定字段为主键
* ColumnInfo  设置列名
* Entity.Index 建立索引
* Entity.ForeignKey 定义外键
* Embedded 定义嵌套对象，将对象中的字段单独存放在表中



### Dao

#### Insert
* 将对象插入表   

```java
 @Dao
public interface MyDao {
      @Insert(onConflict = OnConflictStrategy.REPLACE)
       public void insertUsers(User... users);
 
    @Insert
    public void insertBothUsers(User user1, User user2);
 
    @Insert
    public void insertUsersAndFriends(User user, List<User> friends);
}
```
 
#### Update
* 更新对象

```java
@Dao
public interface MyDao {
    @Update
    public void updateUsers(User... users);
}
```

#### Delete
* 删除对象

```java
@Dao
public interface MyDao {
    @Delete
    public void deleteUsers(User... users);
}
```

#### Query

##### 简单查询
```java
@Dao
public interface MyDao {
    @Query("SELECT * FROM user")
    public User[] loadAllUsers();
}
```

##### 带参数查询
```java
@Dao
public interface MyDao {
    @Query("SELECT * FROM user WHERE age > :minAge")
    public User[] loadAllUsersOlderThan(int minAge);
    
    @Query("SELECT * FROM user WHERE age BETWEEN :minAge AND :maxAge")
    public User[] loadAllUsersBetweenAges(int minAge, int maxAge);
 
    @Query("SELECT * FROM user WHERE first_name LIKE :search "
           + "OR last_name LIKE :search")
    public List<User> findUserWithName(String search);

    //参数也可以是集合
    @Query("SELECT first_name, last_name FROM user WHERE region IN (:regions)")
    public List<NameTuple> loadUsersFromRegions(List<String> regions);
}
```

* Room会使用:minAge去匹配方法中的minAge参数

##### 返回子集
* 只要结果的字段可以和返回的对象匹配，Room允许返回任何的Java对象

```java
public class NameTuple {
    @ColumnInfo(name="first_name")
    public String firstName;
 
    @ColumnInfo(name="last_name")
    public String lastName;
}

@Dao
public interface MyDao {
    @Query("SELECT first_name, last_name FROM user")
    public List<NameTuple> loadFullName();
}
```

##### 可观察的查询
* Room可以返回LiveData类型 和 RxJava2的Flowable类型



### 类型转换器
* 使用TypeConverter来定义转换函数

```java
public class Converters {
    @TypeConverter
    public static Date fromTimestamp(Long value) {
        return value == null ? null : new Date(value);
    }
 
    @TypeConverter
    public static Long dateToTimestamp(Date date) {
        return date == null ? null : date.getTime();
    }
}

@Database(entities = {User.java}, version = 1)
@TypeConverters({Converter.class})
public abstract class AppDatabase extends RoomDatabase {
    public abstract UserDao userDao();
}

```

### 数据库迁移
* 使用Migration类来实现从指定版本到指定版本的数据迁移

```java
Room.databaseBuilder(getApplicationContext(), MyDb.class, "database-name")
        .addMigrations(MIGRATION_1_2, MIGRATION_2_3).build();
 
static final Migration MIGRATION_1_2 = new Migration(1, 2) {
    @Override
    public void migrate(SupportSQLiteDatabase database) {
        database.execSQL("CREATE TABLE `Fruit` (`id` INTEGER, "
                + "`name` TEXT, PRIMARY KEY(`id`))");
    }
};
 
static final Migration MIGRATION_2_3 = new Migration(2, 3) {
    @Override
    public void migrate(SupportSQLiteDatabase database) {
        database.execSQL("ALTER TABLE Book "
                + " ADD COLUMN pub_year INTEGER");
    }
};
```