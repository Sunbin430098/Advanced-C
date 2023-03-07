## C语言进阶

<!-- g++编译调试流程：(单文件编程)
    新建文件夹打开之后，直接将.vscode文件夹粘贴进去即可
    完整流程：
        tasks.json是编译器，里面除了MinGW64路径之外其余都不用改
            终端->配置默认生成任务->g++
        launch.json是调试器，同样只需要改MinGW64路径
            运行->添加配置->g++
        注意：路径不要有中文
    快捷按钮：右上角三角按钮(运行加调试)


cmake编译调试流程：(多文件编程)
    建立：
        ctrl+shift+p->cmake q(cmake快速开始)->输入项目名称->excutable
        ->界面最下方编译即可生成项目的build完整目录
    基础模板：
        cmake_minimum_required (VERSION 2.8.12)#规定cmake的最低版本要求
        project(Cmake_test)#项目的名称，不一定和你的文件夹名称一样
        set(CMAKE_CXX_COMPILER "g++")#设置c++编译器
        
        include_directories(${PROJECT_SOURCE_DIR}/include)#添加头文件的搜索路径
        add_executable(demo ${SrcFiles})#设置可执行文件的名称[${SrcFiles}替换为main函数文件+源文件] 
    快捷按钮：编译是最下栏build，运行是三角，调试是中间的乌龟 -->
