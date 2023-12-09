#### 参考
> [免费的多端同步云笔记 — Obsidian](https://post.smzdm.com/p/aqxrgprv/)
> [官方文档](https://publish.obsidian.md/help-zh/%E7%94%B1%E6%AD%A4%E5%BC%80%E5%A7%8B)
> [Obsidian利用群晖NAS进行多端同步](https://www.xiaohongshu.com/explore/62b51fa20000000001025354)
> [地表最强笔记软件—配合NAS实现了我的多端异地同步](https://post.smzdm.com/p/am8m9n64/)

#### webdav-server
1、在群晖套件中心安装webdav-server套件
![[Pasted image 20230325000016.png]]
2、启用套件端口访问（我给群晖换了https证书所以只启用了https端口）
![[Pasted image 20230325000058.png]]
4、在群晖的控制面板-共享文件夹菜单新建一个共享文件夹，用来保存obsidian通过webdav同步过来的文件
![[Pasted image 20230325001310.png]]

#### openwrt
因为我家里的群晖在openwrt下，openwrt有公网ip，所以需要在openwrt上配置端口映射让外网可以访问webdav-server的端口
![[Pasted image 20230325000323.png]]
配置完成后，就可以在公网的 **“https://域名:映射端口”** 地址访问webdav服务了

#### obsidian安装和仓库设置
1、官网下载[bosidian](https://obsidian.md/) 并安装

2、启动后选择一个文件夹作为本地仓库，默认位置在C盘document目录，建议换到其他盘

3、先不要着急写文档，打开左下角设置-第三方插件，搜索remote，找到插件remotely save并安装，随后启动插件（这个插件用来将本地仓库文件夹同步到webdav服务上，新安装的obsidian打开第三方插件设置时会有安全提醒，关闭即可）
![[Pasted image 20230325000808.png]]

4、安装后的插件会出现在设置页的左侧列表，打开插件设置，修改固定位置的值
![[Pasted image 20230325001131.png]]
	4.2位置选择webdav服务
	4.3位置填写webdav映射到公网的地址，假设openwrt映射了5006端口，则这里的地址为 **“https://host:5006/Obsidian”**，路径末尾的Obsidian就是上面我们在控制面板新建的共享文件夹名称
	4.4位置填写群晖用户名和密码，如果群晖有多个用户，要注意填写的用户必须具有对应共享文件夹的读写权限
	4.5位置在其他都填好后点击检查，如果都填写正确，将提示连接成功
![[Pasted image 20230325001710.png]]

#### 其他设置项
1、新标签的默认视图和默认打开方式
![[Pasted image 20230325002018.png]]
2、删除文件的方式、新建笔记位置、链接类型、附件保存方式等设置
![[Pasted image 20230325002129.png]]
3、同步间隔、是否启动后立即同步一次、是否同步特殊前缀文件和文件夹等
![[Pasted image 20230325002308.png]]

#### 使用技巧

##### 图片和文件的链接
1、当复制了图片或者其他格式文件到剪贴板后，在obsidian编辑器按下ctrl+v粘贴键，将自动把文件或图片复制到附件目录下，并在文档内插入链接符号
2、如果使用qq或者微信的截图快捷键截图后，也可以直接粘贴到文档，同样会自动保存图片到附件目录并插入链接符号

##### 编辑技巧
1、新建文档后，点击右上角书本样的切换按钮，可以在阅读视图和编辑视图之间切换
![[Pasted image 20230325002929.png]]
2、ctrl+左键点击切换按钮则不是切换而是在新标签页打开

3、在文档任意位置按下 **"[ ["** 两个符号，可以唤起快捷链接菜单，支持链接到其他文档、附件等
![[Pasted image 20230325003308.png]]