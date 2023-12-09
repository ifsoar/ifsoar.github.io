地址：[http://www.nohttp.net/](http://www.nohttp.net/)

  

我的demo：

1、添加依赖

1.1、jar文件

[下载Jar包 [含源码，274k]](https://github.com/yanzhenjie/NoHttp/blob/master/Jar/nohttp1.0.6-include-source.jar?raw=true)

[下载Jar包 [不含源码，147k]](https://github.com/yanzhenjie/NoHttp/blob/master/Jar/nohttp1.0.6.jar?raw=true)

1.2、compile

compile 'com.yolanda.nohttp:nohttp:1.0.6'

2、权限

-

-

-

-

-

3、请求

  

String请求

// String 请求对象

Request request = NoHttp.createStringRequest(url, requestMethod);

  

Json请求

- // JsonObject

- Request request = NoHttp.createJsonObjectRequest(url, reqeustMethod);

- ...

- // JsonArray

- Request request = NoHttp.createJsonArrayRequest(url, reqeustMethod);

  

Bitmap请求

  

- Request request = NoHttp.createImageRequest(url, requestMethod);

  

  

添加参数

- Request request = ...

- request.add("name", "yoldada");// String类型

- request.add("age", 18);// int类型

- request.add("sex", '0')// char类型

- request.add("time", 16346468473154); // long类型

- ...

  

请求需要添加到队列中才能启动

- RequestQueue requestQueue = NoHttp.newRequestQueue();

- // 或者传一个并发值，允许三个请求同时并发

- // RequestQueue requestQueue = NoHttp.newRequestQueue(3);

- // 发起请求

- requestQueue.add(what, request, responseListener);

  

异步请求结果监听使用responseListener,重写OnResponseListener接口

  

@Override

public void onStart(int what) {

  

}

  

@Override

public void onSucceed(int what, Response response) {

switch (what) {

case queryWhat_String: {

String string = (String) response.get();

Toast.makeText(this, string, Toast.LENGTH_SHORT).show();

}

break;

}

}

  

@Override

public void onFailed(int what, Response response) {

  

}

  

@Override

public void onFinish(int what) {

  

}  
  

  

同步请求

- Request request = ...

- Response response = NoHttp.startRequestSync(request);

- if (response.isSucceed()) {

- // 请求成功

- } else {

- // 请求失败

- }

  

4、大文件下载

发起请求

- //下载文件

- downloadRequest = NoHttp.createDownloadRequest...

- // what 区分下载Fd

- // downloadRequest 下载请求对象

- // downloadListener 下载监听

- downloadQueue.add(0, downloadRequest, downloadListener);

  

暂停或停止下载

  

- downloadRequest.cancel();

  

下载进度监听

  

- private DownloadListener downloadListener = new DownloadListener() {

- @Override

- public void onStart(int what, boolean resume, long preLenght, Headers header, long count) {

- // 下载开始

- }

- @Override

- public void onProgress(int what, int progress, long downCount) {

- // 更新下载进度

- }

- @Override

- public void onFinish(int what, String filePath) {

- // 下载完成

- }

- @Override

- public void onDownloadError(int what, StatusCode code, CharSequence message) {

- // 下载发生错误

- }

- @Override

- public void onCancel(int what) {

- // 下载被取消或者暂停

- }

- };