(http://www.mingw.org/)

  

-   从官网([http://www.mingw.org/](http://www.mingw.org/))下载MinGW的安装程序
-   安装整个过程都比较简答, 有一点需要注意一下, 就是在选择安装组件的时候除了必须的base和core组件外, 还提供了其他语言的compiler(比如compiler for Objective-C), 用不上的话可以不用勾选.
-   安装完成之后还需要配置环境变量  
    MINGW_HOME=MinGW安装路径  
    再将%MINGW_HOME%\bin加到PATH变量中
-   编写C/C++的helloworld程序并进行编译和测试

```
//c语言版helloworld

// file: hello.c

#include

void main() {

printf("Hello World!");

}
```

编译

```
> gcc hello.c -o c.exe

> c.exe

Hello World!
```

  
```
//c++版

// file: hello.cpp

#include

using namespace std;

main() {

cout << "Hello World!" << endl;

return 0;

}
```

编译

```
> g++ hello.cpp -o cpp.exe

> cpp.exe

Hello World!
```