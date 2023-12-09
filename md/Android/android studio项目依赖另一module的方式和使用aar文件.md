**1、创建新module**

菜单栏 → File → new → new module
![[Pasted image 20230331104436.png]]
选择android library
![[Pasted image 20230331104441.png]]

起个名字再选择上下限版本，点击finish

**2、项目依赖新的module**

在项目主module（一般是app）的gradle.build文件中增加compile project**（注意那个冒号：）**
```
dependencies {

compile fileTree(include: ['*.jar'], dir: 'libs')

compile 'com.android.support:appcompat-v7:25+'

compile project(':soarcoreview')

}
```

  

**3、android library module 开发中需要注意的问题**

怎么删除module？

并不能直接删除一个module，你需要打开project structure
![[Pasted image 20230331104507.png]]

先用鼠标在下面modules的列表中选中要删除的module，然后点上面的红色横杠，确认删除
![[Pasted image 20230331104513.png]]

你会发现目录结构中的module还在，但是变成了普通的文件夹，这时候就可以直接按delete或者右键删除了

  

图片怎么引用？为什么没有mipmap目录？

不知道为什么module中没有mipmap目录，所以我暂时是将图片都放在drawable目录下，这样就可以在xml文件中引用了，但是抱歉，没有代码智能提示
![[Pasted image 20230331104519.png]]

为什么在module中不能使用R.xxx.xxx的形式引用有些资源？

在开发的过程中发现，java文件中没有办法用R.drawable.xxx的形式引用图片资源，很简单,打开File → settings
![[Pasted image 20230331104524.png]]

勾选图示的选项，会将资源也编译进去（原因很简单，对于单独的module工程，as默认不去并联compile）
![[Pasted image 20230331104528.png]]

重新build项目，发现可以使用R引用了

  

**4、生成aar文件**

构建第三方依赖库的时候推荐使用aar代替jar包，因为aar中引用资源文件更加方便，生成方式也很简单，只需要build项目，在module的build → outputs 目录下就会生成aar包文件
![[Pasted image 20230331104535.png]]

**5、引用aar文件**

首先将aar文件放到要引用的项目的libs目录中，然后在app的gradle.build文件中添加如下内容，刷新项目即可
![[Pasted image 20230331104538.png]]