1、添加引用  
```
compile’org.greenrobot:greendao:3.0.1’  
compile’org.greenrobot:greendao-generator:3.0.0’
```

2、在module的build.grade中添加配置  
```
buildscript {  
repositories {  
mavenCentral()  
}  
dependencies {  
classpath ‘org.greenrobot:greendao-gradle-plugin:3.0.0’  
}  
}
```
3、在app的build.grade中添加配置  
```
apply plugin: ‘org.greenrobot.greendao’
```

4、在app的build.grade的根下添加配置（需要自定义生成目录时）  
```
greendao {  
	schemaVersion 1//指定数据库schema版本号，迁移等操作会用到  
	daoPackage ‘com.soar.isit.greendao.gen’//通过gradle插件生成的数据库相关文件的包名，默认为你的entity所在的包名  
	targetGenDir ‘src/main/java’//自定义生成数据库文件的目录了，可以将生成的文件放到我们的java目录中，而不是build中，这样就不用额外的设置资源目录了  
}
```

5、创建一个实体类  
```java
@Entity //代表需要被greendao注解解析的实体  
public class Entity_User {  
	@Id //代表自增主键，类型必须是Long  
	private Long id; //注意这里的类型必须是大写的Long  
	private String username; //默认被解析的字段  
	@Transient //代表不需要创建字段的零时变量  
	private int temp; //不会创建该字段  
}
```

6、点击菜单栏的build project，会发现目录下生成了一些文件,同时在我们的实体中新增了很多代码  
![[Pasted image 20230331103851.png]]
![[Pasted image 20230331103856.png]]

7、执行初始化数据库操作  
```java
DaoMaster.DevOpenHelper openHelper = new DaoMaster.DevOpenHelper(context, “test.db”);  
Database database = openHelper.getWritableDb();  
DaoMaster daoMaster = new DaoMaster(database);  
daoSession = daoMaster.newSession();
```

获取UserDao对象：
```java
mUserDao = MyApplication.getInstances().getDaoSession().getUserDao();
```

简单的增删改查实现：

1.  增

```java
mUser = new User((long)2,”anye3”);  
mUserDao.insert(mUser);//添加一个
```

1.  删
```java
mUserDao.deleteByKey(id);
```

1.  改
```java
mUser = new User((long)2,”anye0803”);  
mUserDao.update(mUser);
```

1.  查
```java
List users = mUserDao.loadAll();  
String userName = “”;  
for (int i = 0; i < users.size(); i++) {  
userName += users.get(i).getName()+”,”;  
}  
mContext.setText(“查询全部数据⇒”+userName);
```

​