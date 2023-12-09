C#中 dll文件放在子目录中的方法

  

VS2012-C#

  

dll文件直接放在程序根目录中（和exe文件一起）比较乱，可以将dll文件放在子文件夹中。步骤如下：

  

1、将dll文件放入子文件夹

  

2、添加引用

  

解决方案资源管理器中，中 工程名或者“引用”上右键，选中添加引用。

  

中引用管理器中，点击浏览，选中子文件夹中的dll文件。
![[Pasted image 20230331105334.png]]
  

  

  

3、修改dll文件的引用属性

  

点击添加成功的引用，将“复制本地”改成false（不然程序运行的时候会将子文件夹下的dll文件复制到根目录中）。
![[Pasted image 20230331105340.png]]
  

4、添加引用的地址，修改config文件

  

在根目录中打开“软件名.exe.config”文件，添加中的语句。

  

其中 probing privatePath 中的地址为子文件的名称。

  

如果有多个子文件夹，两个地址用“;”隔开，如
```
 <?xml version="1.0" encoding="utf-8" ?>
  <configuration>
    <startup>
         <supportedRuntime version="v4.0" sku=".NETFramework,Version=v4.5" />
     </startup>
 
     <runtime>
     <assemblyBinding xmlns="urn:schemas-microsoft-com:asm.v1">
     <probing privatePath="lib"/>
    </assemblyBinding>
    </runtime>

 </configuration>
```