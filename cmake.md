##CMAKE

### 1. error LNK2019: 无法解析的外部符号<br>
编译时出现这种问题，一般是`XXX.lib`的库文件没有设置正确，或者库文件本身有问题。<br>
**note:** 尤其是库文件路径或者cmake变量名的大小写容易粗心写错！！！
### 2. 不允许 dllimport 静态数据成员 的定义<br>
解决方式：使用`__declspec(dllimport)`定为导出类。例如：
    ```
    class __declspec(dllimport) CTest
    {
    }
    ```
另外，有时还会可能遇到unsloved symbol XX,此时使用__declspec(dllimport)修饰，因为静态成员如果不import，是不能够被编译器从lib文件里找到的<br>
**note**：以上问题都是从static变量导出问题的 ，由于导出类的修饰错误导致，需要正确使用__declspec(dllexport) 和 __declspec(dllimport)<br>
对于dll本身来讲，修饰应该是__declspec(dllexport)，而对于调用者来讲，应该是__declspec(dllimport),即生成导出，使用导入<br>
为了方便程序的开发，不用分别写出dll工程的头文件和使用dll工程的头文件，头文件可以写为如下形式：
```
#define OS_API_IMPORT __declspec(dllimport)
      #define OS_API_EXPORT __declspec(dllexport)

      #ifdef BUILD_DLL
      #define OS_API OS_API_EXPORT //如果是生成dll工程，那么导出
      #else
      #define OS_API OS_API_IMPORT //如果是生成使用dll的工程，那么导入
      #endif 

      class OS_API A{static int a;}
```
同时需要在dll工程属性下设置预处理器定义BUILD_DLL.<br>
CMAKE使用`add_definitions(-DNAME)`预定义宏。如果需要选项控指则配合option使用，例如：
```
OPTION(USE_MACRO
  "Build the project using macro"
  OFF)

 IF(USE_MACRO)
  add_definitions("-DUSE_MACRO")
 endif(USE_MACRO)
```
运行构建项目的时候可以添加参数控制宏的开启和关闭．<br>
开启：　`cmake 　-DUSE_MACRO＝on ..`<br>
关闭：　`cmake 　-DUSE_MACRO＝off ..`<br>


参考链接: 
- [https://www.cnblogs.com/darknightsnow/archive/2012/09/25/2701389.html](https://www.cnblogs.com/darknightsnow/archive/2012/09/25/2701389.html)<br>
- [https://blog.csdn.net/dongjideyu/article/details/79267683](https://blog.csdn.net/dongjideyu/article/details/79267683)

### 3. 第三方库
使用第三方库只需要配置 `(.h)`头文件路径和链接`(.lib)`静态库文件即可，以opencv344为例说明如下：<br>
- 配置`(.h)`头文件路径
```
SET(OPENCV_INCLUDE_DIRS ${PROJECT_SOURCE_DIR}/3rdParties/opencvWin32/include ${PROJECT_SOURCE_DIR}/3rdParties/opencvWin32/include/opencv ${PROJECT_SOURCE_DIR}/3rdParties/opencvWin32/include/opencv2)

INCLUDE_DIRECTORIES(${OPENCV_INCLUDE_DIRS})
```
- 链接`(.lib)`静态库文件

```
 SET(OPENCV_LIBRARIES ${PROJECT_SOURCE_DIR}/3rdParties/opencvWin32/x86/vc14/lib/opencv_world344d.lib)

 ADD_LIBRARY(someNameLib XXXll1.cpp XXX2.cpp)
 ADD_LIBRARY(someNameDll SHARED XXX1.cpp XXX2.cpp)
 ADD_EXECUTABLE(someNameExe XXX1.cpp XXX2.cpp)

 TARGET_LINK_LIBRARIES(someNameLib someNameDll someNameExe ${OPENCV_LIBRARIES})
```
或者

```
 SET(OPENCV_LIBRARIES_DIR ${PROJECT_SOURCE_DIR}/3rdParties/opencvWin32/x86/vc14/lib)
 LINK_DIRECTORIES(${OPENCV_LIBRARIES_DIR})

 ADD_LIBRARY(someNameLib XXXll1.cpp XXX2.cpp)
 ADD_LIBRARY(someNameDll SHARED XXX1.cpp XXX2.cpp)
 ADD_EXECUTABLE(someNameExe XXX1.cpp XXX2.cpp)

 TARGET_LINK_LIBRARIES(someNameLib someNameDll someNameExe opencv_world344d)
```

### 4. MSVC编译器选项:指定要使用的主机和目标体系结构

x86：主机计算机体系结构为x86，输出（目标）为x86。<br>
amd64_x86：主机计算机体系结构为x64，输出（目标）为x86。<br>
amd64：主机计算机体系结构为x64版本，输出（目标）为x64。<br>
x86_amd64：主机计算机体系结构为x86或x64版本，输出（目标）为x64。<br>
**note:** 如果使用 x86_amd64，则通常是在一个x86机器上开发的，你希望在x64上创建运行natively的x64文件。 你也可以在x64计算机上使用这里选项，但是你的编译器将在WOW64仿真下运行。<br>
参考链接: 
- [https://docs.microsoft.com/zh-cn/cpp/build/building-on-the-command-line?view=vs-2019](https://docs.microsoft.com/zh-cn/cpp/build/building-on-the-command-line?view=vs-2019)<br>
- [https://kb.kutu66.com/visual-c++/post_1521640](https://kb.kutu66.com/visual-c++/post_1521640)

### 5.在生成.dll文件的同时生成.lib文件
```
# export .lib file (import library) when building dynamic library under Windows
if(WIN32)
       set(CMAKE_WINDOWS_EXPORT_ALL_SYMBOLS ON)
endif()
```

### 6.通配符的使用
例如：在安装中使用：
```
file(GLOB INCFiles "${PROJECT_SOURCE_DIR}/*.h")
MESSAGE(STATUS ${INCFiles})
install(FILES ${INCFiles}
       DESTINATION include/${PROJECT_NAME})
```

### 7.复制指定文件到指定目录
- 方式1  add_custom_command
```
# copy 3rdparty dynamic library to /bin 
if(WIN32)
       # For win, the expected dll files should locate in the same folder with .exe file 
       # cpoy 3rdparty dll library to the path of executable file
       add_custom_command(TARGET test_kalman_tracking POST_BUILD 
       COMMAND ${CMAKE_COMMAND} -E copy_if_different
       "${CMAKE_SOURCE_DIR}/3rdparty/opencv/library/win/opencv_world412d.dll" 
       "${CMAKE_SOURCE_DIR}/3rdparty/hungarian/library/win/hungarian.dll"              
       $<TARGET_FILE_DIR:test_kalman_tracking>)
endif()
```

### 8. 编译依赖：即编译文件A需要先编译文件B
在CMakeLIsts.txt文件中，添加
```
if(编译A)
   set(编译B ON)
endif()
```
例如：tracking依赖于detection
```
if(BUILD_TRACKING)
       set(BUILD_DETECTION ON)
endif()

if(BUILD_DETECTION)
       add_subdirectory(detection)
       message(STATUS "Build module detection")
endif()

if(BUILD_TRACKING)
       set(BUILD_DETECTION ON)
       add_subdirectory(tracking)
       message(STATUS "Build module tracking")
endif()

```

### 9. 设置编译器选项
两种方式：
- `add_compile_options`命令添加的编译选项针对所有的编译器（包括c和c++）<br>
例如：
`add_compile_options(-std=c++11)`说明代码中使用了c++11提供的相关特性，需要编译器支持c++11
- set `CMAKE_CXX_FLAGS`或`CMAKE_C_FLAGS`分别针对c++和c<br>
例如：
`set(CMAKE_CXX_FLAGS "-std=c++11")`

### 10. 指定c++标准
```
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED True)#ON应该也可以
```

### 11.生成路径设置说明
- 以下设置在使用MSVC编译器时会自动在后面添加`debug`或`release`子目录
```
 # set library output path
 set(LIBRARY_OUTPUT_PATH ${CMAKE_BINARY_DIR}/lib)
 # set executable file output path
 set(EXECUTABLE_OUTPUT_PATH ${CMAKE_BINARY_DIR}/bin)
```
**note:** 如果不进行上面的生成路径的设置则生成在当前目录下（即`${CMAKE_BINARY_DIR}`）,同样的MSVC编译器会添加`debug`或`release`子目录，并将生成的文件放在子目录中
<br>**在以上设置的基础上**可通过添加下面的设置去掉`debug`或`release`子目录
```
if(WIN32)
       if(CMAKE_BUILD_TYPE STREQUAL "Debug")
              set(CMAKE_RUNTIME_OUTPUT_DIRECTORY_DEBUG ${CMAKE_BINARY_DIR}/bin)
              set(CMAKE_LIBRARY_OUTPUT_DIRECTORY_DEBUG ${CMAKE_BINARY_DIR}/lib)
              set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY_DEBUG ${CMAKE_BINARY_DIR}/lib)
       else()
              set(CMAKE_RUNTIME_OUTPUT_DIRECTORY_RELEASE ${CMAKE_BINARY_DIR}/bin)
              set(CMAKE_LIBRARY_OUTPUT_DIRECTORY_RELEASE ${CMAKE_BINARY_DIR}/lib)
              set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY_RELEASE ${CMAKE_BINARY_DIR}/lib)
       endif()
endif()
```

### 12. install路径设置
一定要设置为绝对路径，一般相对路径不行
`cmake -A x64 -DCMAKE_INSTALL_PREFIX=绝对路径 ..`

### 13. cmake常用指令含义
<br>`cmake .. `  生成makefile
<br> `cmake --build .` 即make  (完成预编译-》编译-》优化=》汇编-》链接)

### 14. 自己不太熟悉的常用命令
1. `aux_source_dir`: 查找指定目录下的所有源文件并将名称保存到指定变量中
2. `add_subdirectory`: 为本项目添加子目录，该目录下的 CMakeLists.txt 文件和源代码会被处理 
3. `terget_link_libratory`: 指明项目所需链接库 
4. `add_library`: 将源文件生成特定类型库
5. `configure_file`: 从 config.h.in 生成配置头文件 config.h ，通过这样的机制，可以预定义一些参数和变量来控制代码的生成
6. `include_directories`: 添加include文件查找目录
7. `target_include_directories`: 为给定的编译目标添加include目录


### 15.常用变量
1. `PROJECT_SOURCE _DIR`:项目顶层目录, 与`<projectname>_SOURCE_DIR` 和 `CMAKE_SOURCE_DIR`指代内容一致
2. `PROJECT_BINARY_DIR`:项目**编译**顶层目录，与 `<projectname>_BINARY_DIR` 和 `CMAKE_BINARY_DIR`指代内容一致
3. `CMAKE_CURRENT_SOURCE_DIR`:当前处理的 CMakeLists.txt 所在的路径




### 注意事项
1. 多个目录，多个源文件：一种直接的处理方式，分别在项目根目录和各子目录分别编写一个 CMakeLists.txt 文件 
2. 设置可选库：例子`SET (EXTRA_LIBS ${EXTRA_LIBS} MathFunctions)` 当未设置`target_link_libraries (demo  ${EXTRA_LIBS})`时，则不链接该库。工程非常大有很多optional的组件的时候，可以让这个编译文件保持干净
3. `config.h.in`: 在文件`config.h.in` 文件编写预定义等设置， CMake 会自动根据 CMakeLists 配置文件中的设置自动生成 config.h 文件
4. 安装目录：修改 `CMAKE_INSTALL_PREFIX` 变量的值来指定文件安装/拷贝到的根目录
5. 将其他平台的项目迁移到 CMake:  
Visual Studio  
    + `vcproj2cmake.rb` 可以根据 Visual Studio 的工程文件（后缀名是 .vcproj 或 .vcxproj）生成 CMakeLists.txt 文件  
    + `vcproj2cmake.ps1` vcproj2cmake 的 PowerShell 版本。  
    + `folders4cmake `根据 Visual Studio 项目文件生成相应的 “source_group” 信息，这些信息可以很方便的在 CMake 脚本中使用。支持 Visual Studio 9/10 工程文件。