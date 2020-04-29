# cmake学习笔记

## 自己不太熟悉的常用命令
1. `aux_source_dir`: 查找指定目录下的所有源文件并将名称保存到指定变量中
2. `add_subdirectory`: 为本项目添加子目录，该目录下的 CMakeLists.txt 文件和源代码会被处理 
3. `terget_link_libratory`: 指明项目所需链接库 
4. `add_library`: 将源文件生成特定类型库
5. `configure_file`: 从 config.h.in 生成配置头文件 config.h ，通过这样的机制，可以预定义一些参数和变量来控制代码的生成
6. `include_directories`: 添加include文件查找目录
7. `target_include_directories`: 为给定的编译目标添加include目录


## 常用变量
1. `PROJECT_SOURCE _DIR`:项目顶层目录, 与`<projectname>_SOURCE_DIR` 和 `CMAKE_SOURCE_DIR`指代内容一致
2. `PROJECT_BINARY_DIR`:项目**编译**顶层目录，与 `<projectname>_BINARY_DIR` 和 `CMAKE_BINARY_DIR`指代内容一致
3. `CMAKE_CURRENT_SOURCE_DIR`:当前处理的 CMakeLists.txt 所在的路径

## 注意事项
1. 多个目录，多个源文件：一种直接的处理方式，分别在项目根目录和各子目录分别编写一个 CMakeLists.txt 文件 
2. 设置可选库：例子`SET (EXTRA_LIBS ${EXTRA_LIBS} MathFunctions)` 当未设置`target_link_libraries (demo  ${EXTRA_LIBS})`时，则不链接该库。工程非常大有很多optional的组件的时候，可以让这个编译文件保持干净
3. `config.h.in`: 在文件`config.h.in` 文件编写预定义等设置， CMake 会自动根据 CMakeLists 配置文件中的设置自动生成 config.h 文件
4. 安装目录：修改 `CMAKE_INSTALL_PREFIX` 变量的值来指定文件安装/拷贝到的根目录
5. 将其他平台的项目迁移到 CMake:  
Visual Studio  
    + `vcproj2cmake.rb` 可以根据 Visual Studio 的工程文件（后缀名是 .vcproj 或 .vcxproj）生成 CMakeLists.txt 文件  
    + `vcproj2cmake.ps1` vcproj2cmake 的 PowerShell 版本。  
    + `folders4cmake `根据 Visual Studio 项目文件生成相应的 “source_group” 信息，这些信息可以很方便的在 CMake 脚本中使用。支持 Visual Studio 9/10 工程文件。