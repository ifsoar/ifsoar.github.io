
### 缘起
之前把家里两台centos的22端口通过openwrt映射到公网，但一直在家里访问没有真正在公网链接过，结果当最近想通过ssh远程回家里看下centos时，finalshell一直报connection reset。因为我确定在局域网下ssh是正常的，怀疑是openwrt端口转发导致的问题，多次调整配置和google无果，退而求其次，寻找其他替代方案，发现了sshwifty。
### 参考
https://roov.org/2020/10/sshwifty-web-ssh-client/
https://blog.csdn.net/wbsu2004/article/details/125178771
https://zhuanlan.zhihu.com/p/578061503

### 是什么
sshwifty是一个开源的远程ssh解决方案，部署成功后，可以通过浏览器网页的形式远程ssh连接部署环境所在局域网的设备，当然连接公网设备更不在话下。比如我用的典型场景是在单位时可以通过sshwifty在线工具，远程ssh回家里的centos或者路由器等设备。

### 限制
> 需要https访问
> 通过nginx等反向代理时需要启用websocket

### 安装步骤

#### docker下安装sshwifty
在docker注册表搜索sshwifty，选择第一项
![[Pasted image 20230407153910.png]]
选中后下载，等待下载完成；
在映像tab页选中下载完成的sshwifty，点击启动，将会开始启动前配置
![[Pasted image 20230407154047.png]]
#### 启动前配置
端口可以修改，如果不冲突的话可以使用默认的8182
![[Pasted image 20230407154213.png]]
环境变量很多，需要关注的只有三个：
> sharedkey 代表访问密码，也就是部署完成后通过网页访问时的验证码，如果留空则没有身份校验，访问网址可以直接进入在线工具，为了防止被人爆破或者猜到局域网结构，建议加上
> listeninterface 代表授权访问的范围，docker镜像默认值0.0.0.0代表不限制
> listenport 代表服务端口，如果修改的话上一步的端口配置中的容器端口也要同步修改

![[Pasted image 20230407154316.png]]
都配置完成后就可以点击完成，随后docker自动启动，通过群晖ip加端口就已经可以在局域网下访问了。

#### 端口映射和反向代理
> 为了能再公网访问，需要添加openwrt的端口映射，因为sshwifty必须通过https访问，所以不能直接将端口映射到9192，需要将openwrt的端口映射到群晖的反向代理端口，再通过群晖的反向代理，代理到8182。前提条件是群晖已经配置了https证书，我使用的是阿里云1年免费的dv证书，具体申请和安装方式百度即可。

首先在openwrt添加一条端口映射规则到群晖，端口号可以自定义但内外端口号要一致。
![[Pasted image 20230407155616.png]]

然后在群晖的反向代理配置界面添加一条规则
![[Pasted image 20230407155725.png]]
具体规则如下，注意选择来源为https
![[Pasted image 20230407155758.png]]
然后在自定义标题下点击新增，增加websocket，因为sshwifty需要使用websocket通信
![[Pasted image 20230407155851.png]]
点击保存
### 访问

现在通过公网浏览器访问公网域名和openwrt映射的端口号，即可访问sshwifty了，注意使用https协议
如果配置了sharedkey，则需要验证身份
![[Pasted image 20230407160043.png]]

点击左上角加号，选择ssh，然后就可以填写需要连接的ssh设备地址了，如果是局域网设备，直接填写设备的局域网ip即可
![[Pasted image 20230407160124.png]]

第一次连接时会要求确认设备指纹
![[Pasted image 20230407160337.png]]