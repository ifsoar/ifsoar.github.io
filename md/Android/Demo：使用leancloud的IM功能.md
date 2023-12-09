1、创建应用

2、在manifest.xml文件中添加如下权限

  

3、添加依赖

3.1、使用jar文件：

下载地址：[https://leancloud.cn/docs/sdk_down.html](https://leancloud.cn/docs/sdk_down.html)
![[Pasted image 20230331103433.png]]

当前最新为：

3.2使用依赖：

compile ('com.android.support:support-v4:21.0.3')

// LeanCloud 基础包

compile ('cn.leancloud.android:avoscloud-sdk:v3.+')

// 推送与实时聊天需要的包

compile ('cn.leancloud.android:avoscloud-push:v3.+@aar'){transitive = true}

// LeanCloud 统计包

compile ('cn.leancloud.android:avoscloud-statistics:v3.+')

// LeanCloud 用户反馈包

compile ('cn.leancloud.android:avoscloud-feedback:v3.+@aar')

// avoscloud-sns：LeanCloud 第三方登录包  
compile ('cn.leancloud.android:avoscloud-sns:v3.+@aar')  
compile ('cn.leancloud.android:qq-sdk:1.6.1-leancloud')  
// 目前新浪微博官方只提供 jar 包的集成方式

// 请手动下载新浪微博 SDK 的 jar 包，将其放在 libs 目录下进行集成

// LeanCloud 应用内搜索包  
compile ('cn.leancloud.android:avoscloud-search:v3.+@aar')

4、在manifest.xml文件中的activity同级增加如下代码

5、自定义Application并重写onCreate方法，并在manifest.xml文件中使用该application

  

@Override

public void onCreate() {

super.onCreate();

//初始化API

AVOSCloud.initialize(this, "J4XsD0NojPKLhTWFQMJY7eIM-gzGzoHsz", "X83CsRugUppboO74myDpB3gX");

//初始化消息处理类

receiverHandler = new IMReceiverHandler();

//注册

AVIMMessageManager.registerDefaultMessageHandler(receiverHandler);

  

}

其中receiverHandler对象为自定义类，继承了AVIMMessageHandler类，用于处理接收到的消息

  

public class IMReceiverHandler extends AVIMMessageHandler {

  

@Override

public void onMessage(AVIMMessage message, AVIMConversation conversation, AVIMClient client) {

if (message instanceof AVIMTextMessage) {

//do-something...

}

}

}

6、在Activity中实现登录和登出动作

6.1、登录：使用一个字符串生成一个client对象，并登录，client对象之间唯一的识别仅仅靠该字符串

  

instance = AVIMClient.getInstance(edittext_username.getText().toString().trim());

instance.open(new AVIMClientCallback() {

@Override

public void done(AVIMClient client, AVIMException e) {

if (e == null) {

text_content.append("登录成功！\n");

} else {

text_content .append("登录失败！失败信息：" + e.getMessage() + "\n");

}

}

  

});

6.2、登出：与登录类似

  

instance = AVIMClient.getInstance(edittext_username.getText().toString().trim());

instance.close(new AVIMClientCallback() {

@Override

public void done(AVIMClient avimClient, AVIMException e) {

if (e == null) {

text_content.

append("登出成功！\n");

MainActivity.this.instance = null;

} else {

text_content.

append("登出失败！失败信息：" + e.getMessage() + "\n");

}

}

});

7、发送消息，假设当前客户端使用test1登录，向test2发送消息，代码如下：

instance.createConversation(Arrays.asList("test2"), "title of this chat", null, new AVIMConversationCreatedCallback() {

@Override

public void done(AVIMConversation avimConversation, AVIMException e) {

if (e == null) {

AVIMTextMessage msg = new AVIMTextMessage();

msg.setText("Hello test2");

avimConversation.sendMessage(msg, new AVIMConversationCallback() {

@Override

public void done(AVIMException e) {

if (e == null) {

text_content.append("发送消息成功!\n");

}

}

});

} else {

}

}

});

  

---

更多使用方式请参考如下地址：[https://leancloud.cn/docs/realtime_v2.html](https://leancloud.cn/docs/realtime_v2.html)