# cmake学习（一）静态库与动态库构建

**(.so)共享库，shared object：节省空间，在运行时去连接，如果执行机器上没有这些库文件就不能执行。**

**(.a)静态库,archive：静态库和程序化为一体，不会分开。**

**通过 ldd命令可以查看一个可执行程序所依赖的的共享库。**

**使用环境变量LD_LIBRARY_DIRECTORY可以指定共享库位置**



**一、编译共享库：**

```cmake
add_library(hello SHARED ${SHARED_LIBRARY})
```

**二、添加静态库**：

```cmake 
add_library(hello STATIC ${STATIC_LIBRARY})
```

> 因为默认规则是不能有相同名字的共享库与静态库，所以当生成静态库时，共享库会被删除，因为只能允许一个名字存在，相同名字的会被替代，所以需要通过set_target_properties()来解决这个问题，例子：

```cmake
set_target_properties(hello_static PROPERTIES OUTPUT_NAME "hello")
```

cmake在构建一个target的时候，会删除之前生成的target，一样是通过设置set_target_properties(hello PROPERTIES CLEAN_DIRECT_OUTPUT 1)来达到目的

**三、动态库的版本号：**

同样是通过set_target_properties()来设置

```cmake
set_target_properties(hello PROPERTIES VERSION 1.2 SOVERSION 1)

#VERSION：动态库版本
#SOVERSION：API版本

#最后生成的结果是：
#libhello.so.1.2
#libhello.so.1->libhello.so.1.2
#libhello.so->libhello.so.1
```

**四、安装：**

```cmake
install(TARGETS hello hello_static LIBRARY DESTINATION lib ARCHIVE DESTINATION lib)
install(TARGETS hello.h DESTINATION include/hello)

#其他常用的属性
#PERMISSIONS：设置权限；
#RATTERN：设置正则表达式
```

 Summary:

```cmake
add_library()：添加一个库，共享库，静态库，模块
set_target_properties()：设置输出名称，版本号，解决相同target被删除的问题
set_target_properties()：与SET功能相对
```

# cmake学习（二）常用变量和常用环境变量

一、变量的引用方式是使用“${}”，在if中，不需要使用方式，直接使用变量名分类

二、自定义变量使用**set(OBJ_NAME xxxx)**，使用时**${OBJ_NAME}**

三、cmake的常用变量：

- **CMAKE_BINARY_DIR,PROJECT_BINARY_DIR,_BINARY_DIR：**

这三个事件的内容一致，如果是共同的，就指的是工程的环境目录，如果是外部的，那么就是工程发生的目录。

- **CMAKE_SOURCE_DIR，PROJECT_ SOURCE _DIR，_ SOURCE _DIR：**

这三个项目内容一致，都指的是工程的环境目录。

- **CMAKE _CURRENT_BINARY_DIR**：外部编译时，指的是目标目录，内部编译时，指的是环境目录

- **CMAKE_CURRENT_SOURCE_IR：** CMakeList.txt**所属**的目录

- **CMAKE_CURRENT_LIST_DIR：** CMakeList.txt的完整路径

- **CMAKE_CURRENT_LIST_LINE：**当前所在的行

- **CMAKE_MODULE_PATH：**如果工程复杂，可能需要写一些cmake模块，这里通过SET指定

- **LIBRARY_OUTPUT_DIR,BINARYOUTPUT_DIR：**库和可执行的最终活动目录

- **PROJECT_NAME**：工程名

四、cmake中调用环境变量

- 1.using $ENV{NAME}或者 SET(ENV{NAME} value)：调用系统环境变量。

- 2.CMAKE_INCLUDE_CURRENT_DIR 等于 

  include_directory(${CMAKE_CURRENT_BINARY_DIR} ${CMAKE_CURRENT_SOURCE_DIR})

五、其他的自带变量

- 1.BUILD_SHARED_LIBS：使用add_library()时设置默认值

- 2.CMAKE_C_FLAGS：为c语言设置编译器

- 3.CMAKE_CXX_FLAGS：为c++语言设置编译器

六、除bug和发布

- 在工程目录下，cmake -DCMAKE__BUILD_TYPE=DEBUG(RELEASE)，再执行make

七、指定编译32位或64位程序

- set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -m32")

- set(CMAKE_CXX_FLAGS“${CMAKE_CXX_FLAGS} -m32”）

# cmake学习（三）常用指令

一、基本指令：

```cmake
include_directories(${includedir}) #-I。
link_directories(${libdir}) #-L
target_link_libraries(helloworld ${linkflags}) #-l
add_difinitions(${cflags}) #-D
```

- 1、add_difinitions：向C/CPP添加宏定义，相当于gcc中的-D，参数之间用空格分割

- 2、add_difinitions(target_name, depend_name)：定义target对其他target的依赖关系

- 3、aux_source_directory(dir VARIBLE)：把目录下的所有源文件保存在变量中，基本用来创建源文件列表

- 4、add_executable：指定目录，生成执行文件

- 5、exec_program：外部调用指令，可移执行任何外部命令，后面加参数，例子如下：

```cmake
exec_program(ls ARGS "*.c" OUTPUT_VARIBLE LS_OUTPUT RETURN_VALUE LS_RVALUE)
if(not LS_RVALUE)
	message(STATUS "xxx")
endif(not LS_RVALUE)

//PS.这里执行ls *.c指令，执行成功的话，返回0。
```

- 6、FILE指令：

```cmake
file(WRITE file_name "content")
file(APPEND file_name "content")
file(READ file_name varible)
file(WRITE file_name "content")
```

- 7、FIND_系列指令：

```cmake
library(name path)：
find_library(Xorg X11 /usr/lib64)
if(not Xorg)
	message(STATUS "no Xorg")
endif(not Xorg)
file(name path)
path(name path)
program(name path)
package([major.minor][QUIET][NO MODULE][[REQUIRED][COMPONTS][componts....]])
```

- PACKAGE( [major.minor][QUIET][NO MODULE][[REQUIRED][COMPONTS][componts....]])

  用来调用放在CMAKE_MODULE_PATH下的Find.cmake模块，也可以自定义Find模块。首先通过set(CMAKE_MODULE_PATH /home/...)来指定位置

- 8、控制指令：

```cmake
if(expression)

else(expression)

endif(expression)

express举例：
否定：空，0，N，NO，OFF，FALSE，NOTFOUND或_NOTFOUND
肯定：COMMAND cmd，EXISTS dir/file，variable MARCHES regex等等
```

# cmake学习（四）模块的使用和自定义模块

find_package

每一个模块都会产生如下变量

- _FOUND

- _INCLUDE_DIR

- _LIBRARY or _LIBRARIES

如果_FOUND为真，把_INCLUDE_DIR加入到include_directories中，_LIBRARY加入到target_link_libraries中。

编写属于自己的FindHello模块：

```cmake
1.find_path(HELLO_INCLUDE_DIR hello.h /usr/include/hello /usr/local/include/hello)
2.find_library(HELLO_LIBRARY_DIR NAMES hello PATH /usr/lib /usr/local/lib)
 if(HELLO_INCLUDE_DIR AND HELLO_LIBRARY)
 set(HELLO_FOUND TRUE)
 endif(HELLO_INCLUDE_DIR)
3.find_package([major.minor][QUIET][NO_MODULE]
[[REQUIRED|COMPONENTS][componets...]])
```

- QUIET参数：去掉输出信息

- REQUIRED参数：共享库是否是工程必须的，如果是必须的，那么找不到

如果在src中想调用hello模块中的内容

- find_package(HELLO)

为了可以让工程找到FindHELLO.cmake

在主工程的CMakeList.txt中，**set(CMAKE_MODULE_PATH ${PROJECT_SOURCE_PATH}/cmake)**

通过设置find_package(HELLO QUIET)可以去掉输出信息