# C语言进阶



## vscode C工程配置

### g++编译调试流程：(单文件编程)

新建文件夹打开之后，直接将.vscode文件夹粘贴进去即可
完整流程：
    tasks.json是编译器，里面除了MinGW64路径之外其余都不用改
        终端->配置默认生成任务->g++
    launch.json是调试器，同样只需要改MinGW64路径
        运行->添加配置->g++
    注意：路径不要有中文
快捷按钮：右上角三角按钮(运行加调试)
(linux以及windows版本编译调试均有，多文件建议使用cmake)



### cmake编译调试流程：(多文件编程)

​    建立：
​        ctrl+shift+p->cmake q(cmake快速开始)->输入项目名称->excutable
​        ->界面最下方编译即可生成项目的build完整目录
​    基础模板：
​        cmake_minimum_required (VERSION 2.8.12)#规定cmake的最低版本要求
​        project(Cmake_test)#项目的名称，不一定和你的文件夹名称一样
​        set(CMAKE_CXX_COMPILER "g++")#设置c++编译器        	

	include_directories(${PROJECT_SOURCE_DIR}/include)#添加头文件的搜索路径
	add_executable(demo ${SrcFiles})#设置可执行文件的名称[${SrcFiles}替换为main函数文件+源文件] 
	快捷按钮：编译是最下栏build，运行是三角，调试是中间的乌龟    



## cmake使用

## １．cmake命令

### a)基本命令

#### cmake_minimum_required(VERSION xxx)	能使用该项目的最低cmake版本

### project(xxx)	设置项目名称(每个项目都需要此调用，并且应在 cmake_minimum_required() 之后立即调用)

### message(STATUS	"xxx")	运行Cmakelists.txt时向终端输出的信息

### b)项目构建的重要函数

### add_executable(head src/xxx  )	可执行文件(可以表示多个，方便更改)	

```cmake
set(SRC_LIST a.cpp b.cpp c.cpp)	  #可以用变量代替多个文件,${}用在引用时,文件后缀是啥无所谓
add_executable(${PROJECT_NAME} ${SRC_LIST})
```

### aux_source_directory	该命令会查找指定目录下的所有源文件，然后将结果存进指定变量名(对大工程更加方便，前提是确定他们公用一个名称)

```cmake
aux_source_directory(.DIR_SRCS)	#查找当前目录下所有源文件，将源文件名字保存到DIR_SRCS变量(.后面的大写变量名)
add_executable(demo ${DIS_SRCS})
```

### add_subdirectory()	指明本项目下包含一个子目录，目的是子目录下的Cmakelists.txt以及源代码也会被处理(经典例子:ROS　catkin_make后的主目录Cmakelists.txt和./src工程里面的Cmakelists.txt)

### target_link_libraries()	添加链接库

```cmake
target_link_libraries(joy_pub
  ${catkin_LIBRARIES}
  ${OpenCV_LIBS}
)#joy_pub是名称，对照之前的demo，第二个参数是要链接的库 
target_link_libraries(use_head
  head#自定义消息或多文件编程时,针对非executable文件，此处要添加自定义头文件的名字
  ${catkin_LIBRARIES}
)
```

### link_directories()	link_libraries()	这两个不常用或被其他组合代替

### include_directories()	为项目添加头文件目录

```cmake
include_directories(${PROJECT_SOURCE_DIR}/include)#cmake

include_directories(include
  ${catkin_INCLUDE_DIRS})#ROS自定义头文件
```

### add_dependencies(head	(else head))	在目标之间添加依赖项，在自定义多文件工程的时候比较重要．参考[详细的介绍](https://blog.csdn.net/BeanGuohui/article/details/120217097)

### add_library()	使用指定的源文件向项目(project)中添加库，通俗理解就是把自定义的.h .c添加为一个有名字(第一个参数)的库，可执行文件的target_link链接到之后即可调用库	

### [上述链接](https://blog.csdn.net/BeanGuohui/article/details/120217097)综合运用了前述几个函数

总体架构为如下，逻辑上hello和world都是非exe，只有main.c可执行

├── CMakeLists.txt// 下面用主CMAKE表示
├── hello
│   ├── CMakeLists.txt		// 下面用HELLOCMAKE表示
│   ├── hello.c
│   └── hello.h
├── main
│   ├── CMakeLists.txt		// 下面用MAINCMAKE表示
│   └── main.c
└── world
    ├── CMakeLists.txt		// 下面用WORLDCMAKE表示
    ├── world.c
    └── world.h

```cmake
##主Cmake
cmake_minimum_required(VERSION 3.5)

add_subdirectory(main)
add_subdirectory(hello)
add_subdirectory(world)

add_dependencies(CmakeDemo hello world)
	#加这个东西的作用:如果按照add_subdirectories的顺序，先编译main再编译hello world
	#但是如果main中有变量或函数在hello和world中定义，程序就会报错比如找不到变量
	#此时加上add_dependencies(),表示先编译head(此处是CmakeDemo)的依赖(即hello world)，再编译main
	#这种函数并不是为了随便选择位置定义变量准备的，应当注意编码规范

#MainCmake
cmake_minimum_required(VERSION 3.5.1)
project(CmakeDemo C)
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY /home/lib)
set(CMAKE_C_STANDARD 99)

add_executable(CmakeDemo main.c)

link_directories(/home/lib)

target_link_libraries(
        CmakeDemo
        hello
        world
)　
#ello和world经过后面的add_library()操作之后变成了两个库，由于没有被添加到路径(即检测不到)，所以需要target_linik链接

#WorldCmake
cmake_minimum_required(VERSION 3.5.1)

set(CMAKE_C_STANDARD 99)

add_library(world STATIC world.c world.h)#STATIC说明设置为静态库，啥都没有的话默认静态库

#Hello
Cmakecmake_minimum_required(VERSION 3.5.1)

set(CMAKE_C_STANDARD 99)

add_library(hello STATIC world.c hello.h)



```

## 2.动态静态库

#### １）理论知识

静态库	STATIC	(.a 	.lib)	＆	动态库	SHARED	(.so	.dll)

PDF文档里为非常详细的说明，理论上very到位,windows操作不用管

[对g++上的库操作命令进行详细说明](https://blog.csdn.net/m0_58367586/article/details/127035200?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522167843759316800184136063%2522%252C%2522scm%2522%253A%252220140713.130102334.pc%255Fall.%2522%257D&request_id=167843759316800184136063&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~first_rank_ecpm_v1~rank_v31_ecpm-2-127035200-null-null.142^v73^control_1,201^v4^add_ask,239^v2^insert_chatgpt&utm_term=%E6%89%93%E5%8C%85%E9%9D%99%E6%80%81%E5%BA%93%E5%91%BD%E4%BB%A4&spm=1018.2226.3001.4187)

### 	a)静态库：可执行文件A在汇编结束后生成A.o，进入链接阶段时，对封库的源文件经过处理后和A.o**一同链接打包(压缩)成为一组文件B，从此对库和A的任何更改在编译的时候都是整个B一起编译

<div align="static_process">
<img src="doc/image/static_process.png" width=60% />
</div>



### 		特点：移植方便，之前的ROS配置都是静态库配置，但是浪费空间，一个库会反复被调用．更改后整体编译浪费内存	

<div align="static_short">
<img src="doc/image/static_short.png" width=60% />
</div>

```cmake
#打包
g++ -c StaticDemo.cpp	# -c:	将文件编译到汇编结束，生成StaticDemo.o
ar -crv libstaticdemo.a	StaticDemo.o	#将StaticDemo.o打包成静态库文件(显然打包是文件类别应当相同，故为.o文件)
																				#ar为gnu归档工具，-r -c(create) -t -v 四种命令，详见上述链接
#使用(假设main.c执行的时候需要调用StaticDemo生成的静态库)
g++ main.cpp -L./xxx	-lstaticdemo	#-L为添加静态库的路径	-l为库名称，此处注意库名称掐头去尾(lib和.a)
```



### 	b)动态库：程序编译时不会连接动态库，在执行的时候载入，因此如果不同程序都需要调用该库，在内存中存入一份即可，同时不管是程序还是库更新量都更少，不会整体编译．

<div align="dynamic">
<img src="doc/image/dynamic.png" width=60% />
</div>

```cmake
#打包
g++ -fpic -c SharedDemo.cpp #生成.oq-fpic即生成代码，方便多个程序调用共享
g++ -shared -o libshareddemo.so ShareDemo.o	#将SharedDemo.o打包成动态库

#使用
g++ main.cpp -L./xxx	-l shareddemo
```

### 	c)一个运行时的重要不同：

#### 		静态库在汇编之后和可执行程序的.o一起打包，运行程序时内存中一定会检测到

#### 		动态库编译时只会检查是否有添加的动态库路径和名字，有即编译通过不会报错

#### 					执行时首先会检查是否加载了动态库(检查和加载是由动态链接器完成的)，但是如果没有调用动态库的函数，库就不会被加载内存，因此会报加载不到或文件不存在的错误(即动态连接器不知道终端指定的-L路径)

#### 		解决：

#### 					修改系统环境变量，将动态库路径添加到环境变量LD_LIBRARY_PATH中

```cmake
gedit ~/.bashrc
export LD_LIBRARY_PATH=${LD_LIBRARY_PATH}:动态库的绝对路径
修改之后source ~/.bashrc
```



### ２)Cmakelists.txt文件配置项目的库

[参考文档](https://blog.csdn.net/weixin_45004203/article/details/125256367?spm=1001.2101.3001.6661.1&utm_medium=distribute.pc_relevant_t0.none-task-blog-2%7Edefault%7EBlogCommendFromBaidu%7ERate-1-125256367-blog-123531206.pc_relevant_vip_default&depth_1-utm_source=distribute.pc_relevant_t0.none-task-blog-2%7Edefault%7EBlogCommendFromBaidu%7ERate-1-125256367-blog-123531206.pc_relevant_vip_default&utm_relevant_index=1)

```cmake
#与动态静态库设置有关的4个Cmake命令
add_library(A	static/shared	B)	#将B生成库，库名字为libA.so或libA.a
																	#注意:.h和.c都要添加,划重点！！！
include_directories(${PROJECT_SOURCE_DIR}/xxx)		#包含头文件路径
target_link_libraries()						#添加链接库，例如target_link_libraries(hello libhello.so)	这与ROS用到的一致
																 #注意，静态库不需要，只有动态库需要加这个命令，由此可知ROS里面执行的时候调用动态库而不是静态库	
																 #注意:.h和.c都要添加,划重点！！！
																 
#对于main.c即可执行文件来说，需要两条必要命令
add_exxcutable()
add_dependencies()
```

一种思路清晰并且比较规范的为Cmakelists.txt添加头加源文件库可以参考[ROS配置头文件和源文件](http://www.autolabor.com.cn/book/ROSTutorials/di-3-zhang-ros-tong-xin-ji-zhi-jin-jie/32-roszhong-de-tou-wen-jian-yu-yuan-wen-jian/322-zi-ding-yi-yuan-wen-jian-diao-yong.html)

```cmake
#库文件(头加源文件)
include_directories(
include
  ${catkin_INCLUDE_DIRS}
)
add_library(head
  include/test_head_src/haha.h
  src/haha.cpp
)
add_dependencies(head ${${PROJECT_NAME}_EXPORTED_TARGETS} ${catkin_EXPORTED_TARGETS})
target_link_libraries(head
  ${catkin_LIBRARIES}
)


#可执行文件
add_executable(use_head src/use_head.cpp)
add_dependencies(use_head ${${PROJECT_NAME}_EXPORTED_TARGETS} ${catkin_EXPORTED_TARGETS})
target_link_libraries(use_head
  head#此处需要添加之前设置的 head 库
  ${catkin_LIBRARIES}
)

```



###  3 )gcc/g++编译文件原理

[参考文档](https://blog.csdn.net/qq_43479892/article/details/127594711?ops_request_misc=&request_id=&biz_id=102&utm_term=%E4%BD%BF%E7%94%A8%E9%9D%99%E6%80%81%E5%BA%93%E5%91%BD%E4%BB%A4&utm_medium=distribute.pc_search_result.none-task-blog-2~all~sobaiduweb~default-1-127594711.142^v73^control_1,201^v4^add_ask,239^v2^insert_chatgpt&spm=1018.2226.3001.4187)

<div align="dynamic">
    <img src="doc/image/gcc_process.png" width=60% />
</div>

<div align="dynamic">
    <img src="doc/image/gcc_process_code.png" width=60% />
</div>



## 扩充知识之git使用

## 1.基本命令

[git与vscode配置](https://blog.csdn.net/m0_51185558/article/details/126181439)

#### git init 		初始化git仓库

#### git config --global user.name “your name”

#### git config --global user.email “email@email.com“

#### git config --global --list	配置账号信息

#### ssh-keygen -t rsa -C “email@email.com"	创建ssh密钥，方便与远程库直连



## 2.git 基本框架

### 	local	介绍

​		本地库 所有git对project的操作都是在本地完成，只有推拉等方面的操作涉及与远程库的联系

### 	remote	介绍

​		远程库	可以视作本地库的一个备份

```
#### git remote add origin git@github.com:xxx.git　#为本地库添加远程仓库连接

#### git remote -v　#查看远程库信息

#### git remote rm origin	#删除当下与远程库的连接

#### git clone git@github.com:xxx.git #克隆项目

#### git commit -a -m “some commit" #向本地库提交更改

#### git push -u origin master　#推到远程仓库
```



## 3.GirKraken 中涉及的可视化git

<div align="kraken gui">
<img src="doc/image/gitkraken.png" width=99% />
</div>

使用可视化界面主要用于操作分之(branch)

LOCAL下面为本地库，添加branch的时候只能在LOCAL添加，添加之后会提示建立同名远程库(origin/xx)

添加之后双击图像进入分支，右侧选择view all files可以操作project，也可以查看path或tree项目架构

对project内容更改之后　在修改区记得stage all changes,添加Commit Message后点击commit即可

此时更改被添加至本地branch或master库(只在当前分支，其他分支看不到更改)

### branch并入操作

pull request 配置可以选择从本地当前库到master或远程库，origin需要在github.com同意验证

但是此操作主要是为开源项目项目做代码贡献

分支合并可以拖动分支条到目的分支或master,实际上是进行git merge



