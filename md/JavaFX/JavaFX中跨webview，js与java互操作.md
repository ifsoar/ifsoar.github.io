### 说明
使用intellij idea开发，javafx版本17.0.2，jdk版本17，使用gradle进行维护
参考[官方webengine文档](https://openjfx.io/javadoc/17/javafx.web/javafx/scene/web/WebEngine.html)
https://blog.csdn.net/qq_33638188/article/details/126421159
https://blog.csdn.net/moakun/article/details/99897203

### 新建项目
1、没什么好说的，新建javafx项目，选择gradle，jdk选17，等待创建完成，gradle部分内容如下
```java
plugins {  
	id 'java'  
	id 'application'  
	id 'org.javamodularity.moduleplugin' version '1.8.12'  
	id 'org.openjfx.javafxplugin' version '0.0.13'  
	id 'org.beryx.jlink' version '2.25.0'  
}  
  
group 'com.example'  
version '1.0-SNAPSHOT'  
  
repositories {  
	mavenCentral()  
}  
  
sourceCompatibility = '11'  
targetCompatibility = '11'  
  
application {  
mainModule = 'com.example.demo'  
mainClass = 'com.example.demo.HelloApplication'  
}  
  
javafx {  
	version = '17.0.2'  
	modules = ['javafx.controls', 'javafx.fxml','javafx.web']  
}  
```
2、如果创建过程中下载依赖失败，大概率是网络问题，给idea设置proxy代理重试即可
![[Pasted image 20230508151750.png]]

3、新版本idea创建的javafx项目，没有默认的运行配置，需要手动添加
![[Pasted image 20230508151916.png]]
选择Edit Configuration，然后选择左上角+号，选择Application
![[Pasted image 20230508152008.png]]
右侧名字可改可不改，java选17，module选到main，选择主类，ok即可
![[Pasted image 20230508152041.png]]

### 引入webview
默认创建的javafx项目没有webview，需要手动添加。  
先在build.gradle文件中添加组件
```java
javafx {  
	version = '17.0.2'  
	modules = ['javafx.controls', 'javafx.fxml','javafx.web']  
}
```
其中的'javafx.web'是新添加的，添加后同步gradle脚本，等待同步完成

同步完成后需要再引入web的module，打开module-info.java
```java
module com.example.demo {  
requires javafx.controls;  
requires javafx.fxml;  
requires javafx.web;  
requires jdk.jsobject;  
  
  
opens com.example.demo to javafx.fxml;  
exports com.example.demo;  
}
```

其中“requires javafx.web;  ”是新添加的，添加该module才能在项目中使用webview组件；如果需要java和js互操作，或者需要将js的console文本内容重定向到java中，则还需要添加“requires jdk.jsobject; ”这个module。

### webview加载网页
```java
//修改ua头  
webView.getEngine().setUserAgent("Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/112.0.0.0 Safari/537.36");

//加载网址
webView.getEngine().load("https://baidu.com");

//webview加载本地html文件
File file = new File("site/test.html");  
try {
	webView.getEngine().load(file.toURI().toURL().toString());  
} catch (MalformedURLException e) {  
	throw new RuntimeException(e);  
}
```

### 互操作
#### 在java中获取到网页中的alert弹窗内容
```java
//该方法可以获取到js的alert弹框信息  
webView.getEngine().setOnAlert(new EventHandler<WebEvent<String>>() {  
@Override  
	public void handle(WebEvent<String> event) {  
		System.out.println("alert:" + event.toString());  
	}  
});
```

#### js调用java方法
通过将一个java对象映射为js对象，随后使用js语句可以调用该java对象中的方法。
1、首先定义一个java类，并创建一个随后会被js调用的方法。
```java
public class JavaBridge {  
	public void hi(String text) {  
		System.out.println("hi msg from js : " + text);  
	}  
}
```
注意：
> 这个类和这个方法都需要是public的；  
> 这个java类所在的module也需要能够被javafx.web这个module访问到。

为了能够让javafx.web能够访问到新建的java类，可以通过两种方式实现。第一种是将类所在的module直接导出，比如新建javafx项目时，默认创建的主module已经默认被exports了，则在该module中新建的类都能够被javafx.web直接访问，可以查看module-info.java文件。第二种方式是主动将所在的module开放给javafx.web，假设新建的类位于com.foo这个module，则可以在module-info.java中添加如下语句将module开放给javafx.web。
```java
	opens com.foo to javafx.web;
```

2、在适当时机（比如controller的initialize方法中）给webview的engine添加状态监听，并在状态改变时执行语句，将java对象映射给js。
```java
webView.
	getEngine().
	getLoadWorker().
	stateProperty().
	addListener(new ChangeListener<Worker.State>() {  
		@Override  
		public void changed(
			ObservableValue<? extends Worker.State> observable, 
			Worker.State oldValue, 
			Worker.State newValue) {  
		//定义一个public类，并将该类映射到js上下文作为一个对象  
		JavaBridge bridge = new JavaBridge();  
		//获取js的window对象  
		JSObject window = (JSObject) webView.getEngine().executeScript("window");  
		//调用jsObject的setMember方法，将bridge对象设置成window的一个属性  
		window.setMember("javaBridge", bridge);
	}  
});
```

3、在js中调用java方法
```html
<body>  
	<h3>跨浏览器调试测试</h3>  
	<button onclick="callJavaFunc()">js调用java</button>  
	<script>  
		function callJavaFunc() {  
			javaBridge.hi('你好，这是来自js的问候');
		}    
	</script>  
</body>
```

#### 将js的console.log重定向到java中
原理很简单，只需要在上一步的基础上，将console.log这个函数修改为调用我们映射的方法。
在上一步的java代码中添加如下内容
```java
webView.
	getEngine().
	getLoadWorker().
	stateProperty().
	addListener(new ChangeListener<Worker.State>() {  
		@Override  
		public void changed(
			ObservableValue<? extends Worker.State> observable, 
			Worker.State oldValue, 
			Worker.State newValue) {  
		//定义一个public类，并将该类映射到js上下文作为一个对象  
		JavaBridge bridge = new JavaBridge();  
		//获取js的window对象  
		JSObject window = (JSObject) webView.getEngine().executeScript("window");  
		//调用jsObject的setMember方法，将bridge对象设置成window的一个属性  
		window.setMember("consoleBridge", bridge);
		//执行js语句，将console.log重定向到一个自定义的匿名js函数，
		//该函数调用了上一步映射的java对象提供的public方法，并传递log日志文本  
		webView.getEngine().executeScript(
			"console.log = function(message){consoleBridge.log(message);}");
	}  
});
```

#### java调用js方法
1、在js中定义需要被调用的方法
```js
function callFromJava(text) {  
	console.log('该js方法被java调用了，' + text);  
}
```

2、通过如下java代码调用js方法
```java
//获取js的window对象  
JSObject window = (JSObject) webView.getEngine().executeScript("window");
//通过jsObject的call方法，可以调用js函数  
window.call("callFromJava", "再次说你好~");
```

#### java获取js属性
1、在js中定义属性
```js
let name = 'soar';
let count = 20;
let info = {
	name:name,
	setName:function(newName){
		//xxx
	}
};
```

2、在java中获取属性
```java
//获取js的window对象  
JSObject window = (JSObject) webView.getEngine().executeScript("window");

String name = window.eval("name");
int count = window.eval("count");
String infoName = window.eval("info.name");
//执行属性中定义的函数，
window.eval(String.format("info.setName('%s')","soar2"));
```