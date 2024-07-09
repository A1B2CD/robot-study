# 0 编译流程如下图：
![alt text](image.png)
- 蓝色虚线表示使用makefile构建项目的过程
- 红色实线表示使用cmake构建项目的过程

# 1. 使用CMake的基本步骤：
1. 编写CMakeLists.txt文件：
+ 在项目的根目录下创建一个名为CMakeLists.txt的文件，这个文件包含了项目的构建规则和配置选项。

2. 配置项目：
+ 在CMakeLists.txt文件中设置项目名称、语言标准、源文件列表、依赖库等信息。

3. 生成构建系统：
+ 使用CMake命令行工具生成构建系统。这通常在构建目录中完成，而不是在源代码目录中，以避免污染源代码：
    ```
    mkdir build && cd build   # 创建并进入构建目录
    cmake ..
    ```
这里的..指的是CMakeLists.txt文件所在的父目录。

4. 编译项目：
+ 使用生成的构建系统编译项目：
    ```
    cmake --build .
    ```
5. 安装项目（可选）：
如果项目配置了安装规则，可以使用以下命令安装：
    ```
    sudo cmake --install .
    ```
6. 运行测试（可选）：
如果项目配置了测试，可以使用以下命令运行：
    ```
    cmake --build . --target test
    ```
7. 打包项目（可选）：
如果需要将项目打包成二进制或源代码包，可以使用CPack：
    ```
    cmake --build . --target package
    ```


## 1.1 例子
### 简单项目
``` cmake
####    简单项目
cmake_minimum_required(VERSION 3.19)
project(StaffManagementSystem)

set(CMAKE_CXX_STANDARD 14)

add_executable(StaffManagementSystem
        Main/StaffManagerSystem_Main.cpp
        Header/workerManager.h Header/worker.h Header/employee.h Header/manager.h Header/boss.h
        Source/workerManager.cpp Source/employee.cpp Source/manager.cpp Source/boss.cpp
        )
```
----
### 复杂一点
```CMake
cmake_minimum_required(VERSION 3.20.0)
project(my_hello)
set(CMAKE_CXX_STANDARD 11)
include_directories(${PROJECT_SOURCE_DIR}/include)
set(EXECUTABLE_OUTPUT_PATH ${PROJECT_SOURCE_DIR}/bin)
# fangfa1
# add_executable(MyExecutable main.cpp other_source.cpp)
# fangfa2
aux_source_directory(${PROJECT_SOURCE_DIR}/src SRC_LIST)
add_executable(    my_hello    ${SRC_LIST})
```
==注意==:
- 从CMake 3.0开始，推荐使用 target_include_directories 命令代替 include_directories 因为后者会全局地修改所有目标的头文件搜索路径，这可能会导致不必要的头文件路径污染。
- 使用 target_include_directories 可以更精确地控制哪些目标应该包含特定的头文件路径。 
-------

# 2 CMakeLists.txt 文件的编写
## 2.1 基本格式 
```CMAKE
cmake_minimum_required(VERSION 3.0)
project(CALC)
add_executable(app add.c div.c main.c mult.c sub.c)
```
+ `cmake_minimum_required`: 指定使用的 cmake 的最低版本
    +   可选，非必须，如果不加可能会有警告
+ `project` : 定义工程名称，并可指定工程的版本、工程描述、web主页地址、支持的语言（默认情况支持所有语言），如果不需要这些都是可以忽略的，只需要指定出工程名字即可。
    ```CMAKE
    # PROJECT 指令的语法是：
    project(<PROJECT-NAME> [<language-name>...])
    project(<PROJECT-NAME>
    [VERSION <major>[.<minor>[.<patch>[.<tweak>]]]]
    [DESCRIPTION <project-description-string>]
    [HOMEPAGE_URL <url-string>]
    [LANGUAGES <language-name>...])
    ```
+ `add_executable`：定义工程会生成一个可执行程序
    `add_executable(可执行程序名 源文件名称或者源文件集合变量)`
    `add_executable(my_hello   ${SRC_LIST })`

## 2.2   SET()
### i 定义变量
#### reason  
源文件需要反复被使用，每次都直接将它们的名字写出来确实是很麻烦，此时我们就需要定义一个变量，将文件名对应的字符串存储起来
#### 语法
```cmake
# SET 指令的语法是：
# [] 中的参数为可选项, 如不需要可以不写
SET(VAR [VALUE] [CACHE TYPE DOCSTRING [FORCE]])
#  VAR：变量名   VALUE：变量值

#-------------------------------------------------------

# 方式1: 各个源文件之间使用空格间隔
# set(SRC_LIST add.c  div.c   main.c  mult.c  sub.c)
# 方式2: 各个源文件之间使用分号 ; 间隔
set(SRC_LIST add.cpp;div.cpp;main.cpp;mult.c;sub.cpp)
add_executable(app  ${SRC_LIST})
```
### ii 指定c++标准
 -  **在 CMakeLists.txt 中通过 set 命令指定**
```cmake
#增加-std=c++11
set(CMAKE_CXX_STANDARD 11)
#增加-std=c++14
set(CMAKE_CXX_STANDARD 14)
#增加-std=c++17
set(CMAKE_CXX_STANDARD 17)
```
- **在执行 cmake 命令的时候指定出这个宏的值**
```cmake
#增加-std=c++11
cmake CMakeLists.txt文件路径 -DCMAKE_CXX_STANDARD=11
#增加-std=c++14
cmake CMakeLists.txt文件路径 -DCMAKE_CXX_STANDARD=14
#增加-std=c++17
cmake CMakeLists.txt文件路径 -DCMAKE_CXX_STANDARD=17
```
### iii 指定输出路径 
```cmake
## 方式1 定义一个变量用于存储一个绝对路径
set(HOME /home/robin/Linux/Sort)  
## 方式2 将拼接好的路径值设置给EXECUTABLE_OUTPUT_PATH宏
set(EXECUTABLE_OUTPUT_PATH ${HOME}/bin) 
```

## 2.3 多源文件-搜索文件  aux_source_directory() or file()
### i 使用场景 多源文件 .cpp
- 如果一个项目里边的源文件很多，在编写CMakeLists.txt文件的时候不可能将项目目录的各个文件一一罗列出来，这样太麻烦也不现实。所以，在CMake中为我们提供了搜索文件的命令，可以使用aux_source_directory命令或者file命令。
### ii aux_source_directory( path A  ) 应用
```cmake
# 1. 查找某个路径下的所有源文件
## aux_source_directory(< dir > < variable >)
### dir：要搜索的目录  variable：将从dir目录下搜索到的源文件列表存储到该变量中
cmake_minimum_required(VERSION 3.0)
project(CALC)
include_directories(${PROJECT_SOURCE_DIR}/include)
# 搜索 src 目录下的源文件
aux_source_directory(${CMAKE_CURRENT_SOURCE_DIR}/src SRC_LIST)
add_executable(app  ${SRC_LIST})
```
### iii file(GLOB A path) 应用
```cmake
# 2. file（当然，除了搜索以外通过 file 还可以做其他事情）。
#file(GLOB/GLOB_RECURSE 变量名 要搜索的文件路径和文件类型)
## GLOB: 将指定目录下搜索到的满足条件的所有文件名生成一个列表，并将其存储到变量中。
## GLOB_RECURSE：递归搜索指定目录，将搜索到的满足条件的文件名生成一个列表，并将其存储到变量中
file(GLOB MAIN_SRC ${CMAKE_CURRENT_SOURCE_DIR}/src/*.cpp)
file(GLOB MAIN_HEAD ${CMAKE_CURRENT_SOURCE_DIR}/include/*.h)

```

## 2.4 包含头文件 .h  - include_directories( path )
```cmake
include_directories(${PROJECT_SOURCE_DIR}/include)
##PROJECT_SOURCE_DIR宏对应的值就是我们在使用cmake命令时，后面紧跟的目录，一般是工程的根目录。
```
## 2.5 制作动态库 静态库
### i 概念
#### i.i 库

- 库的目的:
 库是代码的集合，通常包含了一系列功能函数、类、宏等，它们被设计为==可重用的==。库可以提供特定的功能，比如数学计算、图形渲染、网络通信等。
 <br/>
- 库的分发:
 ==库文件可以独立于应用程序分发.== 开发者可以只发布库文件，而不必发布源代码或应用程序本身.其他开发者可以在他们的项目中链接这些库，从而使用库提供的功能，而不需要自己重新实现这些功能。
 <br/>
- API和ABI：
库开发者需要定义清晰的API（应用程序编程接口）和ABI（应用程序二进制接口），以便第三方开发者知道如何使用库，并确保兼容性。
 <br/>
-----

####  i.ii 静态库

- 文件扩展名为 `.a` 在Unix-like系统，静态库名字分为三部分：`lib`+库名字+`.a`,`.lib` 在Windows系统
- 定义 : 静态库包含了一组编译过的源代码的二进制文件集合，它们在程序编译时被整合到最终的可执行文件中
<br/>
- 编译过程：在编译阶段，程序员选择将库中的哪些对象文件链接到最终的可执行文件中。
<br/>
- 优点：
    - 简化部署：因为库的代码已经被包含在可执行文件中，所以不需要在运行时分发库文件。
    - 性能：由于在编译时链接，可能减少运行时的内存占用和提高加载速度。
- 缺点：
    - 增加可执行文件大小：因为库的代码被复制到了每个使用它的可执行文件中。
    - 更新困难：更新库可能需要重新编译所有依赖它的可执行文件。

- **创建** :
    - 格式
        ```cmake
        add_library(库名称 STATIC 源文件1 [源文件2] ...) 
        ```
    - 例子
        - 目录
        ```shell
            ├── build
            ├── CMakeLists.txt
            ├── include           # 头文件目录
            │   └── head.h
            ├── main.cpp          # 用于测试的源文件
            └── src               # 源文件目录
                ├── add.cpp
                ├── div.cpp
                ├── mult.cpp
                └── sub.cpp
        ```
        - CMakeLsit.txt
        ```cmake
        cmake_minimum_required(VERSION 3.0)
        project(CALC)
        include_directories(${PROJECT_SOURCE_DIR}/include)
        file(GLOB SRC_LIST "${CMAKE_CURRENT_SOURCE_DIR}/src/*.cpp")
        add_library(calc STATIC ${SRC_LIST})
        ```
----------------------------------------------------------------

#### i.iii 动态库
- 文件扩展名为 `.so` 在Unix-like系统，动态库名字分为三部分：`lib`+库名字+`.so`, `.dll` 在Windows系统 
- 定义：动态库在程序运行时被加载，==它们允许多个程序共享同一份库代码，而不需要将代码复制到每个可执行文件中。==
- 编译过程：在编译阶段，程序员生成一个动态库文件，并在程序中使用特殊的链接指令来标识库的依赖关系，==但实际的库代码直到程序运行时才会被加载。==
- **优点：**
    - 减少磁盘占用：多个程序可以共享同一份库文件，减少磁盘空间的占用。
    - 易于更新：更新动态库不需要重新编译使用它的程序，只需要替换库文件即可。
- **缺点：**
    - 部署复杂：需要确保运行时库文件的可访问性，可能需要配置环境变量或指定库文件的路径。
    - 可能增加启动时间：程序启动时需要加载库，可能会稍微增加启动时间
- **创建:** 
    - 格式
    `add_library(库名称 SHARED 源文件1 [源文件2] ...) `
    - 例子
        ```cmake
        cmake_minimum_required(VERSION 3.0)
        project(CALC)
        include_directories(${PROJECT_SOURCE_DIR}/include)
        file(GLOB SRC_LIST "${CMAKE_CURRENT_SOURCE_DIR}/src/*.cpp")
        add_library(calc SHARED ${SRC_LIST})
        ```

### ii 对比
| 对比   |   创建方法 |  加载        
|-------|-----------| --|
|静态库| `add_library(库名称 STATIC 源文件1 [源文件2] ...) ` |静态库会在生成可执行程序的链接阶段被打包到可执行程序中，所以可执行程序启动，静态库就被加载到内存中了。|
|动态库| `add_library(库名称 SHARED 源文件1 [源文件2] ...) ` |动态库在生成可执行程序的链接阶段不会被打包到可执行程序中，当可执行程序被启动并且调用了动态库中的函数的时候，动态库才会被加载到内存|

### iii 问题
#### 1.什么时候用静态库和动态库,什么时候不需要?
例如在src/main.cpp 和 src/helper.cpp 这些源文件通常直接编译进最终的可执行文件，而不是被编译成库.
需要:  代码重用 ; 模块化设计 ;  动态加载   ;  API提供

### iv 指定输出路径
#### 方式 1 适用于动态库  set( EXECUTABLE_OUTPUT_PATH   )
- 对于生成的库文件来说和可执行程序一样都可以指定输出路径。由于在Linux下生成的动态库默认是有执行权限的，所以可以按照生成可执行程序的方式去指定它生成的目录：
    ```cmake
    cmake_minimum_required(VERSION 3.0)
    project(CALC)
    include_directories(${PROJECT_SOURCE_DIR}/include)
    file(GLOB SRC_LIST "${CMAKE_CURRENT_SOURCE_DIR}/src/*.cpp")
    # 设置动态库生成路径
    set(EXECUTABLE_OUTPUT_PATH ${PROJECT_SOURCE_DIR}/lib)
    ####
    add_library(calc SHARED ${SRC_LIST})
    ```  
#### 方式 2  set( LIBRARY_OUTPUT_PATH  _ )
- 由于在Linux下生成的静态库默认不具有可执行权限，所以在指定静态库生成的路径的时候就不能使用EXECUTABLE_OUTPUT_PATH宏了，而应该==使用LIBRARY_OUTPUT_PATH==，这个宏对应静态库文件和动态库文件都适用。
    ```cmake
    cmake_minimum_required(VERSION 3.0)
    project(CALC)
    include_directories(${PROJECT_SOURCE_DIR}/include)
    file(GLOB SRC_LIST "${CMAKE_CURRENT_SOURCE_DIR}/src/*.cpp")
    # 设置动态库/静态库生成路径
    set(LIBRARY_OUTPUT_PATH ${PROJECT_SOURCE_DIR}/lib)
    # 生成动态库
    #add_library(calc SHARED ${SRC_LIST})
    # 生成静态库
    add_library(calc STATIC ${SRC_LIST})
    ```
## 2.6 包含库文件 链接库文件
### i. 链接静态库
- 制作静态库 `add_library(库名称 STATIC 源文件1 [源文件2] ...)` -一个静态库文件libcalc.a
- 也有其他方式 [通过命令制作使用静态链接库](https://subingwen.cn/linux/library/#1-1-%E7%94%9F%E6%88%90%E9%9D%99%E6%80%81%E9%93%BE%E6%8E%A5%E5%BA%93)
- 链接静态库 `link_libraries(<static lib> [<static lib>...])`
    - 2. 假设静态库名为 libmystaticlib.a `
target_link_libraries(my_target PRIVATE mystaticlib)`
    - 可以是全名 `libxxx.a`
    - 也可以是掐头`（lib）`去尾`（.a）`之后的名字 `xxx`
    - 找不到静态库  -- link_directories(< lib path >)
- CMakeLists.txt
    ```cmake
    cmake_minimum_required(VERSION 3.0)
    project(CALC)
    # 搜索指定目录下源文件
    file(GLOB SRC_LIST ${CMAKE_CURRENT_SOURCE_DIR}/src/*.cpp)
    # 包含头文件路径
    include_directories(${PROJECT_SOURCE_DIR}/include)
    # 包含静态库路径
    link_directories(${PROJECT_SOURCE_DIR}/lib)
    # 链接静态库
    link_libraries(calc)
    add_executable(app ${SRC_LIST})
    ```

### ii. 链接动态库
- 制作动态库
    - 其他方式 [linux-静态库和动态库](https://subingwen.cn/linux/library/)
- 链接动态库
    ```cmake    
    target_link_libraries(
    <target> #可以是可执行文件、静态库、动态库或接口库。
    <PRIVATE|PUBLIC|INTERFACE> <item>... 
    [<PRIVATE|PUBLIC|INTERFACE> <item>...]...)
    # 表示要链接到 <target> 的项目。<item> 可以是库目标的名称、库文件的路径，或者是库的别名
    ```
- 动态库的链接具有==传递性==
动态库 A 链接了动态库B、C，动态库D链接了动态库A，此时动态库D相当于也链接了动态库B、C，并可以使用动态库B、C中定义的方法
    ```cmake
    target_link_libraries(A B C)
    target_link_libraries(D A)
    ```
- 注意
    - ==在cmake中指定要链接的动态库的时候，应该将命令写到生成了可执行文件之后：==
    ```cmake
    cmake_minimum_required(VERSION 3.0)
    project(TEST)
    file(GLOB SRC_LIST ${CMAKE_CURRENT_SOURCE_DIR}/*.cpp)
    # 添加并指定最终生成的可执行程序名
    add_executable(app ${SRC_LIST})
    # 指定可执行程序要链接的动态库名字
    target_link_libraries(app pthread)
    ``` 
- 链接第三方库
    - 在生成可执行程序之前，通过命令指定出要链接的动态库的位置，指定静态库位置使用的也是这个命令：`link_directories(path)`
    - CMakeLists.txt 
    ```cmake
    cmake_minimum_required(VERSION 3.0)
    project(TEST)
    file(GLOB SRC_LIST ${CMAKE_CURRENT_SOURCE_DIR}/*.cpp)
    # 指定源文件或者动态库对应的头文件路径
    include_directories(${PROJECT_SOURCE_DIR}/include)
    # 指定要链接的动态库的路径
    link_directories(${PROJECT_SOURCE_DIR}/lib)
    # 添加并生成一个可执行程序
    add_executable(app ${SRC_LIST})
    # 指定要链接的动态库
    target_link_libraries(app pthread calc)
    ```
## 2.7 日志
### - 格式
```cmake
message([STATUS|WARNING|AUTHOR_WARNING|FATAL_ERROR|SEND_ERROR] "message to display" ...)
# STATUS ：非重要消息
# WARNING：CMake 警告, 会继续执行
# AUTHOR_WARNING：CMake 警告 (dev), 会继续执行
# SEND_ERROR：CMake 错误, 继续执行，但是会跳过生成的步骤
# FATAL_ERROR：CMake 错误, 终止所有处理过程
```
```cmake
# 输出一般日志信息
message(STATUS "source path: ${PROJECT_SOURCE_DIR}")
# 输出警告信息
message(WARNING "source path: ${PROJECT_SOURCE_DIR}")
# 输出错误信息
message(FATAL_ERROR "source path: ${PROJECT_SOURCE_DIR}")
```


## 2.8 变量操作 
### 1. 源文件不在同一个目录中  - 追加
- 第一步 `file`  各个目录下的源文件进行搜索
- 第二步 字符串拼接
    - 法1 `set(变量名1 ${变量名1} ${变量名2} ...)`
        - 例子
        ```cmake
        cmake_minimum_required(VERSION 3.0)
        project(TEST)
        set(TEMP "hello,world")
        file(GLOB SRC_1 ${PROJECT_SOURCE_DIR}/src1/*.cpp)
        file(GLOB SRC_2 ${PROJECT_SOURCE_DIR}/src2/*.cpp)
        # 追加(拼接)
        set(SRC_1 ${SRC_1} ${SRC_2} ${TEMP})
        message(STATUS "message: ${SRC_1}")
        ```
    - 法2 使用list 拼接 `list(APPEND <list> [<element> ...])`
        ```cmake
        cmake_minimum_required(VERSION 3.0)
        project(TEST)
        set(TEMP "hello,world")
        file(GLOB SRC_1 ${PROJECT_SOURCE_DIR}/src1/*.cpp)
        file(GLOB SRC_2 ${PROJECT_SOURCE_DIR}/src2/*.cpp)
        # 追加(拼接)
        list(APPEND SRC_1 ${SRC_1} ${SRC_2} ${TEMP})
        message(STATUS "message: ${SRC_1}")
        ```
### 2. 有些源文件并不是我们所需要的  - 字符串移除
####  `list(REMOVE_ITEM <list> <value> [<value> ...]) `
- 例子: 将main.cpp从搜索到的数据中剔除出去
    ```cmake
    cmake_minimum_required(VERSION 3.0)
    project(TEST)
    set(TEMP "hello,world")
    file(GLOB SRC_1 ${PROJECT_SOURCE_DIR}/*.cpp)
    # 移除前日志
    message(STATUS "message: ${SRC_1}")
    # 移除 main.cpp  SRC_1
    list(REMOVE_ITEM SRC_1 ${PROJECT_SOURCE_DIR}/main.cpp)
    # 移除后日志
    message(STATUS "message: ${SRC_1}")
    ```
- 注意 
    -可以看到，在第8行把将要移除的文件的名字指定给list就可以了。但是一定要注意通过 file 命令搜索源文件的时候得到的是文件的绝对路径（在list中每个文件对应的路径都是一个item，并且都是绝对路径），那么==在移除的时候也要将该文件的绝对路径指定出来才可以，否是移除操作不会成功==。

## 2.9 宏定义
### 1. 格式  add_definitions(-D宏名称)
### 2. CMakeLists.txt 
```CMake
cmake_minimum_required(VERSION 3.0)
project(TEST)
# 自定义 DEBUG 宏
add_definitions(-DDEBUG)
add_executable(app ./test.c)
```
### 3. 作用
- 在进行程序测试的时候，我们可以在代码中添加一些宏定义，通过这些宏来控制这些代码是否生效，如下所示：
    ```c++
    #include <stdio.h>
    #define NUMBER  3

    int main()
    {
        int a = 10;
    #ifdef DEBUG
        printf("我是一个程序猿, 我不会爬树...\n");
    #endif
        for(int i=0; i<NUMBER; ++i)
        {
            printf("hello, GCC!!!\n");
        }
        return 0;
    }

    ```
-  上述代码中并没有定义这个宏,我们可通过CMakeList

# 3. 预定义宏 

|宏	|功能|
|--|---|
|PROJECT_SOURCE_DIR	|使用cmake命令后紧跟的目录，一般是工程的根目录
PROJECT_BINARY_DIR	|执行cmake命令的目录
CMAKE_CURRENT_SOURCE_DIR|	当前处理的CMakeLists.txt所在的路径
CMAKE_CURRENT_BINARY_DIR|	target 编译目录
EXECUTABLE_OUTPUT_PATH|	重新定义目标二进制可执行文件的存放位置
LIBRARY_OUTPUT_PATH|	重新定义目标链接库文件的存放位置
PROJECT_NAME	|返回通过PROJECT指令定义的项目名称
CMAKE_BINARY_DIR|	项目实际构建路径，假设在build目录进行的构建，那么得到的就是这个目录的路径



# 4 大项目  -嵌套
- 目录
    ```shell
    $ tree
    .
    ├── build
    ├── calc
    │   ├── add.cpp
    │   ├── CMakeLists.txt #子节点
    │   ├── div.cpp
    │   ├── mult.cpp
    │   └── sub.cpp
    ├── CMakeLists.txt #根节点  父节点
    ├── include
    │   ├── calc.h
    │   └── sort.h
    ├── sort
    │   ├── CMakeLists.txt #子节点
    │   ├── insert.cpp
    │   └── select.cpp
    ├── test1
    │   ├── calc.cpp
    │   └── CMakeLists.txt #子节点
    └── test2
        ├── CMakeLists.txt  #子节点
        └── sort.cpp

    ```
    - 根节点`CMakeLists.txt`中的变量 全局有效
    - 父节点`CMakeLists.txt`中的变量 可以在子节点中使用
    - 子节点`CMakeLists.txt`中的变量 只能在当前节点中使用
- 确定父子关系 `add_subdirectory(source_dir [binary_dir] [EXCLUDE_FROM_ALL])`
    - `source_dir` 指定了CMakeLists.txt源文件和代码文件的位置，其实就是指定子目录
    - `[binary_dir]` 指定了输出文件的路径，一般不需要指定，忽略即可。
    - `EXCLUDE_FROM_ALL`  在子路径下的目标默认不会被包含到父路径的ALL目标里，并且也会被排除在IDE工程文件之外。用户必须显式构建在子路径下的目标。
<br/>

## 4.1 根目录 CMake构建 `定义全局变量`和`添加子目录`
    ```cmake
    cmake_minimum_required(VERSION 3.0)
    project(test)
    # 定义变量
    # 静态库生成的路径
    set(LIB_PATH ${CMAKE_CURRENT_SOURCE_DIR}/lib)
    # 测试程序生成的路径
    set(EXEC_PATH ${CMAKE_CURRENT_SOURCE_DIR}/bin)
    # 头文件目录
    set(HEAD_PATH ${CMAKE_CURRENT_SOURCE_DIR}/include)
    # 静态库的名字
    set(CALC_LIB calc)
    set(SORT_LIB sort)
    # 可执行程序的名字
    set(APP_NAME_1 test1)
    set(APP_NAME_2 test2)
    # 添加子目录
    add_subdirectory(calc)
    add_subdirectory(sort)
    add_subdirectory(test1)
    add_subdirectory(test2)
    ```
<br/>

## 4.2  calc目录 Cmake构建  :  生成静态库
    ```cmake
    cmake_minimum_required(VERSION 3.0)
    project(test)
    # 定义变量
    # 静态库生成的路径
    set(LIB_PATH ${CMAKE_CURRENT_SOURCE_DIR}/lib)
    # 测试程序生成的路径
    set(EXEC_PATH ${CMAKE_CURRENT_SOURCE_DIR}/bin)
    # 头文件目录
    set(HEAD_PATH ${CMAKE_CURRENT_SOURCE_DIR}/include)
    # 静态库的名字
    set(CALC_LIB calc)
    set(SORT_LIB sort)
    # 可执行程序的名字
    set(APP_NAME_1 test1)
    set(APP_NAME_2 test2)
    # 添加子目录
    add_subdirectory(calc)
    add_subdirectory(sort)
    add_subdirectory(test1)
    add_subdirectory(test2)
    ```
    - 第3行aux_source_directory：搜索当前目录（calc目录）下的所有源文件
    - 第4行include_directories：包含头文件路径，HEAD_PATH是在根节点文件中定义的
    - 第5行set：设置库的生成的路径，LIB_PATH是在根节点文件中定义的
    - 第6行add_library：生成静态库，静态库名字CALC_LIB是在根节点文件中定义的
<br/>

## 4.3  sort 目录 CMake 构建    -生成动态库
```cmake
cmake_minimum_required(VERSION 3.0)
project(SORTLIB)
aux_source_directory(./ SRC)
include_directories(${HEAD_PATH})
set(LIBRARY_OUTPUT_PATH ${LIB_PATH})  
add_library(${SORT_LIB} SHARED ${SRC})  #生成动态库 SORT_LIB在根节点定义
```
## 4.4  test1 目录 CMake 构建
```cmake
cmake_minimum_required(VERSION 3.0)
project(CALCTEST)
aux_source_directory(./ SRC)
include_directories(${HEAD_PATH}) #指定头文件路径，HEAD_PATH变量是在根节点文件中定义的
link_directories(${LIB_PATH}) #
link_libraries(${CALC_LIB}) # 指定可执行程序要链接的静态库，CALC_LIB变量是在根节点文件中定义的
set(EXECUTABLE_OUTPUT_PATH ${EXEC_PATH}) #指定可执行程序生成的路径，EXEC_PATH变量是在根节点文件中定义的
add_executable(${APP_NAME_1} ${SRC}) #add_executable：生成可执行程序，APP_NAME_1变量是在根节点文件中定义的

```

## 4.5 test2 目录 CMake 构建
```cmake
cmake_minimum_required(VERSION 3.0)
project(SORTTEST)
aux_source_directory(./ SRC)
include_directories(${HEAD_PATH}) #包含头文件路径
set(EXECUTABLE_OUTPUT_PATH ${EXEC_PATH}) #指定可执行程序生成的路径
link_directories(${LIB_PATH}) #指定可执行程序要链接的动态库的路径
add_executable(${APP_NAME_2} ${SRC})
target_link_libraries(${APP_NAME_2} ${SORT_LIB}) #指定可执行程序要链接的动态库的名字
```
# 5 流程控制

# 6 引用-资料
- [C++ CMake入门和进阶(一)：使用CMake编译项目--CSDN](https://blog.csdn.net/JackSparrow_sjl/article/details/119609956)

- [cmake开发手册中文版--GITEE](https://gitee.com/open-php/cmake-manual-cn)  

- [CMake 保姆级教程](https://subingwen.cn/cmake/CMake-primer/)