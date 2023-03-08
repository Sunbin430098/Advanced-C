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



## 扩充知识之git使用

## 1.基本命令

[１]: https://blog.csdn.net/m0_51185558/article/details/12618143９	"git基本配置"

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

​	![](/home/ubuntu/图片/GitKraken.png)

使用可视化界面主要用于操作分之(branch)

LOCAL下面为本地库，添加branch的时候只能在LOCAL添加，添加之后会提示建立同名远程库(origin/xx)

添加之后双击图像进入分支，右侧选择view all files可以操作project，也可以查看path或tree项目架构

对project内容更改之后　在修改区记得stage all changes,添加Commit Message后点击commit即可

此时更改被添加至本地branch或master库(只在当前分支，其他分支看不到更改)

### branch并入操作

pull request 配置可以选择从本地当前库到master或远程库，origin需要在github.com同意验证

但是此操作主要是为开源项目项目做代码贡献

分支合并可以拖动分支条到目的分支或master,实际上是进行git merge



