参考

[http://blog.csdn.net/chaozhung_no_l/article/details/49929177](http://blog.csdn.net/chaozhung_no_l/article/details/49929177)  

---

1、监听home键

定义  
```java
private final BroadcastReceiver homeReceiver = new BroadcastReceiver() {
	final String SYS_KEY = "reason";
	// 标注下这里必须是这么一个字符串值
	final String SYS_HOME_KEY = "homekey";
	// 标注下这里必须是这么一个字符串值
	@Override
	public void onReceive(Context context, Intent intent) {
		String action = intent.getAction();
		if (action.equals(Intent.ACTION_CLOSE_SYSTEM_DIALOGS)) {
			String reason = intent.getStringExtra(SYS_KEY);
			if (reason != null && reason.equals(SYS_HOME_KEY)) {
				Log.i("TT", "##################home键监听");
				Toast.makeText(MainActivity.this, "aaaa",
				Toast.LENGTH_sHORT).show();
				Intent intentw = new Intent(Intent.ACTION_MAIN);
				intentw.addCategory(Intent.CATEGORY_HOME);
				intentw.setClassName("android",
				"com.android.internal.app.ResolverActivity");
				startActivity(intentw);
			}
		}
	}
};
```


在onCreate注册  
```java
IntentFilter homeFilter = new IntentFilter(
	Intent.ACTION_CLOSE_SYSTEM_DIALOGS);
registerReceiver(homeReceiver, homeFilter);
``` 

2、弹出系统的桌面选择框

标准Android使用  
```
//无需操作，当系统安装有超过一个launcher应用时，会默认弹出
```

大部分定制rom  
```
//在上面的onReceive中已经实现
```

华为（未测试）  
```java
Intent paramIntent = new Intent("android.intent.action.MAIN");
paramIntent.setComponent(new ComponentName("com.huawei.android.internal.app", "com.huawei.android.internal.app.HwResolverActivity"));
paramIntent.addCategory("android.intent.category.DEFAULT");
paramIntent.addCategory("android.intent.category.HOME");
mContext.startActivity(paramIntent);
```
  

3、跳转系统设置  

```java
startActivity(new Intent(Settings.ACTION_SETTINGS));
```

标准Android使用以下方式可以直接跳转系统的默认桌面设置界面，定制rom不行  

```java
startActivity(new Intent(Settings.ACTION_HOME_SETTINGS));
```

4、将应用设置为系统桌面  
```xml
<activity android:name=".MainActivity"
    android:launchMode="singleInstance">
    <intent-filter>
        <action android:name="android.intent.action.MAIN" />
        <category android:name="android.intent.category.LAUNCHER" />
        <category android:name="android.intent.category.HOME"/>
        <category android:name="android.intent.category.DEFAULT" />
        <category android:name="android.intent.category.MONKEY" />
    </intent-filter>
</activity>
```


国产定制rom需要在系统设置中修改默认桌面设置，活着像上面那样监听home键然后弹出系统的桌面选择框