1、添加依赖

```
dependencies {

compile 'org.litepal.android:core:1.5.1'

}
```

  

2、在asssets中创建litepal.xml文件
![[Pasted image 20230331103602.png]]
```xml
<?xml version="1.0" encoding="utf-8"?>
<litepal>
     <!-- 数据库名字 -->
     <dbname value="demo" />
     <!-- 数据库的版本号 -->
     <version value="1" />
     <!-- 要映射的实体类路径 For example:
     <list>
          <mapping class="com.test.model.Reader" />
          <mapping class="com.test.model.Magazine" />
     </list>
     -->
     <list>
     </list>
     <!-- //可以选择存储数据库/内部或者外部 //默认internal -->
     <storage value="external" />
</litepal>
```

3、在androidmanifest.xml文件中使用LitePalApplication或者让自己的Application继承LitePalApplication

```java
public class MyApplication extends LitePalApplication {
	...
}
```

  

4、新建实体类，继承dataSupport（不要使用id字段，如果一定要用必须用大写Long类型定义）
```java
public class PO_Account extends DataSupport implements Serializable {

@Column(unique = true)//唯一

private String oid;

private String groupOid;

private String name;

private String username;

private String password;

private long createTime;

private long updateTime;

@Column(ignore = true)//忽略

private List remarkList;

@Column(ignore = true)//忽略

private List tagList;

@Column(ignore = true)//忽略

private PO_Group group;

//...记得添加set和get方法

}
```

  

5、增删改查

存储操作

```java
Movie movie=new Movie();

movie.setName("3Diots");

movie.setDirector("Indians");

//这一步就是将一条记录存储进数据库中

movie.save();
```

删除记录
```java
//删除数据库中movie表的所有记录

DataSupport.deleteAll(Movie.class);

//删除数据库movie表中id为1的记录

DataSupport.deleteAll(Movie.class,1);

//删除数据库movie表中duration大于3500的记录

DataSupport.deleteAll(Movie.class, "duration > ?" , "3500");
```
修改记录

方法一

```java
//第一步，查找id为1的记录

Movie movie = DataSupport.find(Movie.class, 1);

//第二步，改变某个字段的值

movie.setPrice(4020f);

//第三步，保存数据

movie.save();
```

方法二
```java
Movie movie=new Movie();

movie.setName("2Diots");

movie.setDirector("某人");

//直接更新id为1的记录

movie.update(1);
```
方法三

```java
Movie movie=new Movie();

movie.setDirector("someone");

//更新所有name为2Diots的记录,将director字段设为someone

movie.updateAll("name = ?", "2Diots");
```

查询记录

```java
//查找movie表的所有记录

List allMovies = DataSupport.findAll(Movie.class);

  

//查找movie表id为1的记录

Movie movie = DataSupport.find(Movie.class,1);

  

//查找name为2Diots的记录,条件查询，以时长作排序

List movies = DataSupport.where("name = ?", "2Diots").order("duration").find(Movie.class);
```