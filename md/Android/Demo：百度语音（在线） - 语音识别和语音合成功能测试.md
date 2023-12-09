0、注册帐号和创建应用

![[Pasted image 20230331102616.png]]
  
![[Pasted image 20230331102627.png]]

![[Pasted image 20230331102631.png]]

1、百度语音识别

下载对应sdk

![[Pasted image 20230331102638.png]]
![[Pasted image 20230331102649.png]]

将压缩包libs下的armeabi文件下下的so和jar包放到新建的android studio项目的libs下
![[Pasted image 20230331102659.png]]

在app的build.gradle下添加配置
![[Pasted image 20230331102706.png]]

在androidManifest.xml文件中添加权限和配置

权限增加如下
```xml
<uses-permission android:name="android.permission.RECORD_AUDIO" />
<uses-permission android:name="android.permission.INTERNET" />
<uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
<uses-permission android:name="android.permission.ACCESS_WIFI_STATE" />
<uses-permission android:name="android.permission.CHANGE_WIFI_STATE" />
<uses-permission android:name="android.permission.READ_PHONE_STATE" />
<uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />

<uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE" />
<uses-permission android:name="android.permission.WRITE_SETTINGS" />
<uses-permission android:name="android.permission.MODIFY_AUDIO_SETTINGS" />
```

application节点下增加如下
```xml
<!-- begin: baidu speech sdk-->
<!-- 请填写应用实际的APP_ID -->
<meta-data android:name="com.baidu.speech.APP_ID" android:value="10027779"/>
<!-- 请填写应用实际的API_KEY -->
<meta-data android:name="com.baidu.speech.API_KEY" android:value="5mGXbKiZy7tILIXXm0b1kBtG"/>
<!-- 请填写应用实际的SECRET_KEY -->
<meta-data android:name="com.baidu.speech.SECRET_KEY" android:value="1tW2HcRjd8epi06ijzObFeNuyKqC2GL6"/>
<service android:name="com.baidu.speech.VoiceRecognitionService" android:exported="false" ></service>
<activity
    android:name="com.baidu.voicerecognition.android.ui.BaiduASRDigitalDialog"
    android:configChanges="orientation|keyboardHidden|screenLayout"
    android:theme="@android:style/Theme.Dialog"
    android:exported="false"
    android:screenOrientation="portrait">
    <intent-filter>
        <action android:name="com.baidu.action.RECOGNIZE_SPEECH" ></action>
        <category android:name="android.intent.category.DEFAULT" ></category>
    </intent-filter>
</activity>
<!-- end : baidu speech sdk-->
```

开发代码

创建speechRecognizer对象
```java
speechRecognizer = SpeechRecognizer.createSpeechRecognizer(this, new ComponentName(this, VoiceRecognitionService.class));

speechRecognizer.setRecognitionListener(this);
```

  

调用以下方法开始录音和识别
```java
private void startASR() {

Intent intent = new Intent();

bindParams(intent);

speechRecognizer.startListening(intent);

}

  

private void bindParams(Intent intent) {

intent.putExtra("sample", 16000);

intent.putExtra("language", "cmn-Hans-CN");

}
```

  

临时识别结果和正式结果会从以下方法回调
```java
@Override
public void onReadyForSpeech(Bundle params) {
	//只有该方法回调后才能开始识别
	Log.d(TAG, "onReadyForSpeech");
}
@Override
public void onBeginningOfSpeech() {
	//开始说话
	Log.d(TAG, "onBeginningOfSpeech");
}
@Override
public void onRmsChanged(float rmsdB) {
	//音量变化
	Log.d(TAG, "onRmsChanged");
}
@Override
public void onBufferReceived(byte[] buffer) {
	Log.d(TAG, "onBufferReceived>>>" + new String(buffer));
}
@Override
public void onEndOfSpeech() {
	//结束说话
	Log.d(TAG, "onEndOfSpeech");
}
@Override
public void onError(int error) {
	//出错
	Log.d(TAG, "onError>>>" + error);
}
@Override
public void onResults(Bundle results) {
	//正式识别结果
	Log.d(TAG, "onResults>>>" + results);
	List<String> list = results.getStringArrayList("results_recognition");
	((EditText) findViewById(R.id.edit_main_input)).setText(list.get(0));
}
@Override
public void onPartialResults(Bundle partialResults) {
	//临时识别结果
	Log.d(TAG, "onPartialResults>>>" + partialResults);
}
@Override
public void onEvent(int eventType, Bundle params) {
	//事件回调
	Log.d(TAG, "onEvent>>>" + eventType + ":" + params);
}
```

  

2、百度语音生成

  

下载对应的sdk
![[Pasted image 20230331102904.png]]
![[Pasted image 20230331102910.png]]

将libs下的so和jar放到libs下
![[Pasted image 20230331102915.png]]

  

增加配置
![[Pasted image 20230331102921.png]]

  

调用如下方法初始化
```java
private void startTTS() {
	// 获取语音合成对象实例
	speechSynthesizer = SpeechSynthesizer.getInstance();
	// 设置context
	speechSynthesizer.setContext(this);
	// 设置语音合成状态监听器
	speechSynthesizer.setSpeechSynthesizerListener(new SpeechSynthesizerListener() {
		@Override
		            public void onSynthesizeStart(String s) {
			Log.d(TAG2, "onSynthesizeStart>>>" + s);
		}
		@Override
		            public void onSynthesizeDataArrived(String s, byte[] bytes, int i) {
			Log.d(TAG2, "onSynthesizeDataArrived>>>" + s);
		}
		@Override
		            public void onSynthesizeFinish(String s) {
			Log.d(TAG2, "onSynthesizeFinish>>>" + s);
		}
		@Override
		            public void onSpeechStart(String s) {
			Log.d(TAG2, "onSpeechStart>>>" + s);
		}
		@Override
		            public void onSpeechProgressChanged(String s, int i) {
			Log.d(TAG2, "onSpeechProgressChanged>>>" + s);
		}
		@Override
		            public void onSpeechFinish(String s) {
			Log.d(TAG2, "onSpeechFinish>>>" + s);
		}
		@Override
		            public void onError(String s, SpeechError speechError) {
			Log.d(TAG2, "onError>>>" + s);
		}
	}
	);
	// 设置在线语音合成授权，需要填入从百度语音官网申请的api_key和secret_key
	speechSynthesizer.setApiKey("5mGXbKiZy7tILIXXm0b1kBtG", "1tW2HcRjd8epi06ijzObFeNuyKqC2GL6");
	// 设置离线语音合成授权，需要填入从百度语音官网申请的app_id
	speechSynthesizer.setAppId("10027779");
	// 设置语音合成文本模型文件
	//        speechSynthesizer.setParam(SpeechSynthesizer.PARAM_TTS_TEXT_MODEL_FILE, "your_txt_file_path");
	// 设置语音合成声音模型文件
	//        speechSynthesizer.setParam(SpeechSynthesizer.PARAM_TTS_SPEECH_MODEL_FILE, "your_speech_file_path");
	// 设置语音合成声音授权文件
	//        speechSynthesizer.setParam(SpeechSynthesizer.PARAM_TTS_LICENCE_FILE, "your_licence_path");
	// 获取语音合成授权信息
	AuthInfo authInfo = speechSynthesizer.auth(TtsMode.MIX);
	// 判断授权信息是否正确，如果正确则初始化语音合成器并开始语音合成，如果失败则做错误处理
	if (authInfo.isSuccess()) {
		speechSynthesizer.initTts(TtsMode.MIX);
		//  speechSynthesizer.speak("百度语音合成示例程序正在运行");
	} else {
		Log.d(TAG2, "authInfo>>>" + authInfo.getTtsError().getDetailCode());
		// 授权失败
	}
}
```


如果授权成功，调用如下代码即可播放合成声音
![[Pasted image 20230331103000.png]]