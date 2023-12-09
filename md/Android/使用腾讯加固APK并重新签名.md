参考
https://itlao5.com/wp/809.html
https://blog.csdn.net/spoling/article/details/107984071
https://wiki.open.qq.com/wiki/%E5%BA%94%E7%94%A8%E5%8A%A0%E5%9B%BA#3._.E5.A6.82.E4.BD.95.E4.BD.BF.E7.94.A8.E4.B9.90.E5.9B.BA.E5.8A.A0.E5.9B.BA.E6.9C.8D.E5.8A.A1

https://cloud.tencent.com/developer/article/1376347?from=article.detail.1135340

  

1、先用as签名生成release包

2、访问腾讯加固上传页面

[https://console.cloud.tencent.com/ms/reinforce/list](https://console.cloud.tencent.com/ms/reinforce/list)

上传apk加固

3、下载加固包，在如下位置执行命令重新加固
```bash
//在Android\Sdk\build-tools\30.0.3路径下执行

.\apksigner.bat sign --ks f:\xxxx\gameboxkey.jks --ks-key-alias mainkey --ks-pass pass:gamebox --key-pass pass:gamebox --out G:\app-steady-sign.apk G:\app-steady.apk

```
--ks为签名密钥

--ks-key-alias

--ks-pass pass：为密码

--key-pass pass：为密码

--out a.apk b.apk，a为签名后，b为未签名的加固包