编译libssh2的方法如下：本次应用都是x64平台

1准备: 需要有zlib     openssl     libssh2（本机下载的文件在D:\data\my_work\GeneralPurposeUnitTestUtility\v1.0\ssh）
	zlib百度在官网下载  

	openssl（https://link.csdn.net/?target=https%3A%2F%2Fwww.libssh2.org%2F）编译libssh2需要安装OpenSSL，这里自己编译库比较复杂，直接安装带库的包比较方便：

	直接从 Win32/Win64 OpenSSL Installer for Windows - Shining Light Productions 下载，注意，不要下载 light 版本，因为 light 版本不带库文件，我下载的是libssh2-1.10.0.tar.gz
	
	 libssh2下载（https://www.libssh2.org/）

2编译依赖库zlib与openssl
	编译zlib:
	打开 适用于 VS 2017 的 x64 本机工具命令提示（vs的64位cmd），cd 进入D:\data\my_work\GeneralPurposeUnitTestUtility\v1.0\ssh\zlib\zlib-1.2.11\contrib\masmx64目录，执行bld_ml64.bat，
	然后会在D:\data\my_work\GeneralPurposeUnitTestUtility\v1.0\ssh\zlib\zlib-1.2.11\contrib\vstudio\vc10下生成sln项目文件，用vs打开，解决方案中包含多个项目，用vs debug生成解决方案，在	ZlibDllDebug目录下找到zlibwapid.dll  zlibwapid.lib，
	编译openssl:
	下载的openssl就是带库文件的包，直接可以用不用编译；

3编译libssh2:
	配置编译：打开D:\data\my_work\GeneralPurposeUnitTestUtility\v1.0\ssh\libssh2-1.10.0\libssh2-1.10.0\win32下的config.mk，修改zlib(include、 lib) openssl(include、lib)的配置情况，本机配置	的	是：
	!if "$(OPENSSLINC)" == ""
	OPENSSLINC=D:\data\my_work\GeneralPurposeUnitTestUtility\v1.0\ssh\Win64OpenSSL-3_0_3\OpenSSL-Win64\include
	!endif

	!if "$(OPENSSLLIB)" == ""
	OPENSSLLIB=D:\data\my_work\GeneralPurposeUnitTestUtility\v1.0\ssh\Win64OpenSSL-3_0_3\OpenSSL-Win64\lib
	!endif

	!if "$(ZLIBINC)" == ""
	ZLIBINC=D:\data\my_work\GeneralPurposeUnitTestUtility\v1.0\ssh\zlib\builded
	!endif

	!if "$(ZLIBLIB)" == ""
	ZLIBLIB=D:\data\my_work\GeneralPurposeUnitTestUtility\v1.0\ssh\zlib\builded
	!endif
	//上面的配置信息也可以在cmake的时候用-D选项添加进去  
	
	编译命令：
	cd D:\data\my_work\GeneralPurposeUnitTestUtility\v1.0\ssh\libssh2-1.10.0\libssh2-1.10.0   //进入目录
	mkdir build   //创建编译目录
	mkdir dll   //创建编译目录
	cd build         //进入编译目录
	cmake -DCMAKE_GENERATOR_PLATFORM=x64 -DENABLE_ZLIB_COMPRESSION=ON -DCRYPTO_BACKEND=OpenSSL -DBUILD_SHARED_LIBS=ON -DCMAKE_INSTALL_PREFIX=./dll --build ..    	//编译命令。 选项-DCMAKE_GENERATOR_PLATFORM=x64 选择目标平台为64位； -DENABLE_ZLIB_COMPRESSION=ON 依赖库zlib的压缩选项的开关打开；-DCRYPTO_BACKEND=OpenSSL 依赖库	选择OpenSSL； -DBUILD_SHARED_LIBS=ON建立动态链接库；-DCMAKE_INSTALL_PREFIX=./dll  将生成的lib dll exe文件安装在./dll目录下；--build . .在上一级目录生成。
	此时在上一级目录，即D:\data\my_work\GeneralPurposeUnitTestUtility\v1.0\ssh\libssh2-1.10.0\libssh2-1.10.0 下生成了sln的解决方案；用vs打开它，升级，里面有很多项目，依然采用debug x64配	置，会在D:\data\my_work\GeneralPurposeUnitTestUtility\v1.0\ssh\libssh2-1.10.0\libssh2-1.10.0\src\Debug目录下生成你不是说libssh2项目的lib dll文件。头文件在根目录下。

4使用
	然后复制我们刚刚编译好的 libssh2 的库文件和所需的头文件。

    	在新建的 项目目录下新建两个目录，一个叫 include，一个叫 lib。
    	复制我们刚才生成好的D:\data\my_work\GeneralPurposeUnitTestUtility\v1.0\ssh\libssh2-1.10.0\libssh2-1.10.0\src\Debug  lib和dll文件到新建项目的 lib 目录下。
    	复制 libssh2-1.10.0\include 目录下所有文件到新建的 项目的 include 目录下。
    	复制 libssh2-1.10.0\win32\libssh2_config.h文件到新建的项目的 include 目录下。
	添加附加包含目录：c++ ----将新建的 项目的 include文件夹包含进来；
	添加附加库目录：链接器--常规--附加库目录--将新建的 lib 目录加进来；
	添加附加依赖库：链接器--输入--附加依赖项--将新建的 lib文件加进来；

	这里项目属性就配置好了，我们到官网找一个测试代码，拷贝到我们的项目里试一下。打开 https://www.libssh2.org/examples，随便找一个例子，我这里找的是 ssh2_exec.c 的列子，选择 ssh2_exec.c 	例子，将里面所有的代码复制出来到我们测试项目的 main.cpp 中，然后修改一下例子中连接服务器的 IP 地址、用户名口令等信息。
	编译项目，提示了如下错误
	在代码的第一行（注意一定是第一行，所有 include 的前面）加上一句 #define _WINSOCK_DEPRECATED_NO_WARNINGS，禁用这个警告
	这是因为我们使用了 Windows socket 库里面的函数，要包含一下 ws2_32.lib 库文件，一样在附加依赖库中，增加 ws2_32.lib
	再次编译，这次编译成功了。

	参考：http://www.guoziweb.com/html/other/2018/06/13/2265.html
		http://www.ilovecpp.com/2017/09/24/build-libssh2/
		https://www.cnblogs.com/17bdw/p/11216448.html
		https://www.jianshu.com/p/3c46bc12ae04
		https://blog.csdn.net/qq_37887537/article/details/120488903
		https://blog.csdn.net/qq_45662588/article/details/122560356
		


