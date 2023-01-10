# CMake Tutorial

[toc]



## Step1. A Basic Strring Point

### Exercise 1. Building a Basic Project

最基本的CMake项目是从单个源代码文件构建的, 一个简单的`CmakeLists.txt`文件至少包含必须的三个命令.

1.   `cmake_minimum_required()` : 任何的CMake项目最顶端的一行命令, 指定使用CMake的最低版本.
2.   `project()`: 设置项目的名称,对每个项目来说是必须的, 应该在`cmake_minimum_required()`之后
3.   `add_executable()`: 告诉CMake使用特定的源代码文件创建可执行程序.

例子: 使用三个命令构建一个基本的项目.

```txt
# TODO 1: Set the minimum required version of CMake to be 3.10
cmake_minimum_required(VERSION 3.10)
# TODO 2: Create a project named Tutorial
project(Tutorial)
# TODO 3: Add an executable called Tutorial to the project
# Hint: Be sure to specify the source file as tutorial.cxx
add_executable(Tutorial tutorial.cxx)

```

1.   运行`cmake`命令来配置项目, 可以先创建一个用于构建项目的目录`mkdir Step1_build`, 进入目录, 执行`cmake ../`
2.   `cmake --build .`在构建目录执行当前命令编译链接项目
3.   在当构建目录下可以看到一个可执行的项目`Tutorial`, 可以运行当前命令测试



### Exercise2-Specifying the C++ Standard

使用`CMAKE_CXX_STANDARD`或者`CMAKE_CXX_STANDARD_REQUIRED`来指定用于构建项目的C++标准

例子: 在源文件中添加C++11特性, 然后更新`CMakeLists.txt`, 来指定需要C++11.

这是源文件中更改的部分, 引入C++11特性:

```cpp
// #include <cstdlib> // TODO 5: Remove this line

  // TODO 4: Replace atof(argv[1]) with std::stod(argv[1])
  // const double inputValue = atof(argv[1]);
  const double inputValue = std::stod(argv[1]);
```



这是`CMakeLists.txt`变换的部分:

```txtt
# TODO 6: Set the variable CMAKE_CXX_STANDARD to 11
#         and the variable CMAKE_CXX_STANDARD_REQUIRED to True
set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED True)
```

然后先通过`cmake ../`和`cmake --build`得到更新的配置和重新编译链接.



### Exercise 3 Adding a Version Number and Configured Header File

在`CMakelists.txt`中定义的变量也能在源代码中访问

情况是这样的, 我希望将`TutorialConfig.h.in`中定义的一些变量复制到`TutorialConfig.h`中, 这些变量的值由`CMakeLists.txt`定义, 在我的`TutorialConfig.h.in`中, 我是这样定义的:

```hpp
// the configured options and settings for Tutorial
// TODO 10: Define Tutorial_VERSION_MAJOR and Tutorial_VERSION_MINOR
 #cmakedefine Tutorial_VERSION_MAJOR @Tutorial_VERSION_MAJOR@
 #cmakedefine Tutorial_VERSION_MINOR @Tutorial_VERSION_MINOR@
```

然后我使用CMake构建, 发现会产生这样一个错误:

```bash
Scanning dependencies of target Tutorial
[ 50%] Building CXX object CMakeFiles/Tutorial.dir/tutorial.cxx.o
/home/cyxb/cmake/Step1/tutorial.cxx: In function ‘int main(int, char**)’:
/home/cyxb/cmake/Step1/tutorial.cxx:15:48: error: ‘Tutorial_VERSION_MINOR’ was not declared in this scope
  std::cout << Tutorial_VERSION_MAJOR << "." << Tutorial_VERSION_MINOR << std::endl;
                                                ^~~~~~~~~~~~~~~~~~~~~~
/home/cyxb/cmake/Step1/tutorial.cxx:15:48: note: suggested alternative: ‘Tutorial_VERSION_MAJOR’
  std::cout << Tutorial_VERSION_MAJOR << "." << Tutorial_VERSION_MINOR << std::endl;
                                                ^~~~~~~~~~~~~~~~~~~~~~
                                                Tutorial_VERSION_MAJOR
CMakeFiles/Tutorial.dir/build.make:62: recipe for target 'CMakeFiles/Tutorial.dir/tutorial.cxx.o' failed
make[2]: *** [CMakeFiles/Tutorial.dir/tutorial.cxx.o] Error 1
CMakeFiles/Makefile2:67: recipe for target 'CMakeFiles/Tutorial.dir/all' failed
make[1]: *** [CMakeFiles/Tutorial.dir/all] Error 2
Makefile:83: recipe for target 'all' failed
make: *** [all] Error 2
```

即,在其中一个变量是未定义的, 然后我查看`TutorialConfig.h`这个文件, 查看复制了哪些变量,结果如下:

```hpp
// the configured options and settings for Tutorial
// TODO 10: Define Tutorial_VERSION_MAJOR and Tutorial_VERSION_MINOR
#define Tutorial_VERSION_MAJOR 1
/* #undef Tutorial_VERSION_MINOR */
```

会发现第二个变量没有定义, 但是根据`CMakeLists.txt`, 第一个和第二个变量都定义了, 并且值是1和0.

如果我将原来`TutorailConfig.h.in`中的定义修改为如下的形式:

```hpp
// the configured options and settings for Tutorial
// TODO 10: Define Tutorial_VERSION_MAJOR and Tutorial_VERSION_MINOR
#define Tutorial_VERSION_MAJOR @Tutorial_VERSION_MAJOR@
#define Tutorial_VERSION_MINOR @Tutorial_VERSION_MINOR@
```

然后重新构建,在`TutorialConfig.h`中会发现量变量都有定义:

```hpp
// the configured options and settings for Tutorial
// TODO 10: Define Tutorial_VERSION_MAJOR and Tutorial_VERSION_MINOR
#define Tutorial_VERSION_MAJOR 1
#define Tutorial_VERSION_MINOR 0
```

这是为什么呢?

后来在网上查,找到一个相似的问题:

[[CMake] #cmakedefine vs #define](https://cmake.org/pipermail/cmake/2011-March/043464.html)



> Hello,
>
> Sorry if this question is too primitive. I am trying to extract the build dir
> into the source code by populating it in a variable in my config.h file.
>
> I do this by adding the following line in config.h.in
>
> #cmakedefine PROJ_BUILD_DIR_CMAKEDEFINE "${PROJECT_BINARY_DIR}"
>
> In the config.h, this produces
>
> /* #undef PROJ_BUILD_DIR_CMAKEDEFINE */
>
>
>
> But if i replace it to this in config.h.in,
>
> #define PROJ_BUILD_DIR_DEFINE "${PROJECT_BINARY_DIR}"
>
> Then the right value is populated in the config.h
>
> #define PROJ_BUILD_DIR_DEFINE "/home/bala/projects/myproj/trunk/builds"
>
> Wondering why didn't the cmakedefine work in this case? Any clues?

因为`#cmakedefine`会被替换为`#define`或者`/*#undef */`, 而在`CMakeLists.txt`中第二个变量的值恰好是0, 可能因为这个原因,被当做没有定义.为了验证这个猜测, 我做了如下是实验:

将`CMakeLists.txt`中第二个变量的值设置为1, 同时将原本定义为`True`的另外一个变量设置为`False`, 然后在`TutorialConfig.h.in`中引入这新的变量, 为了对照, 我分别使用`#cmakedefine`和`#define`引入:

```hpp
#cmakedefine Tutorial_VERSION_MINOR  ${Tutorial_VERSION_MINOR}
#cmakedefine CMAKE_CXX_STANDARD ${CMAKE_CXX_STANDARD}
#cmakedefine CMAKE_CXX_STANDARD_REQUIRED ${CMAKE_CXX_STANDARD_REQUIRED}
#define cmake_cxx_standard_required ${CMAKE_CXX_STANDARD_REQUIRED}
```

然后重新编译构建, 得到的`TutorialConfig.h`如下:

```hpp
#define Tutorial_VERSION_MINOR  1
#define CMAKE_CXX_STANDARD 11
/* #undef CMAKE_CXX_STANDARD_REQUIRED */
#define cmake_cxx_standard_required False
```

结果和猜测差不多, 因为`#cmakedefine`可能会将空, 零值, `False`等当做变量没有定义来处理, 而`#define`则是直接复制值.

在帖子的最后, 作者任务`#cmakedefine`对于依赖于条件`#ifdef`的情况下非常有用, 其它情况下使用`#define`直接复制值更好.



上面练习的一个解决办法如下:

1.   首先在`CMakeLists.txt`中定义版本相关的变量的值
2.   然后通过`config_file`来配置复制的输入`TutorialConfig.h.in`和输出文件`TuotrialConfig.h`
3.   使用`traget_inclue_directories`来包含`${PROJECT_BINARY_DIR}`
4.   在`TutorialConfig.h.in`中定义要复制的变量
5.   在`tutorial.cxx`中引入包含的头文件, 就是`config_file`中输出的文件,然后使用变量

涉及三个文件,

`CMakeLists.txt`如下:

```txt
# TODO 7: Set the project version number as 1.0 in the above project command
project(Tutorial VERSION 1.1)
# TODO 8: Use configure_file to configure and copy TutorialConfig.h.in to
#         TutorialConfig.h
configure_file(TutorialConfig.h.in TutorialConfig.h )

# TODO 9: Use target_include_directories to include ${PROJECT_BINARY_DIR}
target_include_directories(Tutorial PUBLIC ${PROJECT_BINARY_DIR})
                                                           
```

`TutorialConfig.h.in`定义的变量:

```hpp
// the configured options and settings for Tutorial
// TODO 10: Define Tutorial_VERSION_MAJOR and Tutorial_VERSION_MINOR
#define Tutorial_VERSION_MAJOR @Tutorial_VERSION_MAJOR@
#cmakedefine Tutorial_VERSION_MINOR  ${Tutorial_VERSION_MINOR}
#cmakedefine CMAKE_CXX_STANDARD ${CMAKE_CXX_STANDARD}
#cmakedefine CMAKE_CXX_STANDARD_REQUIRED ${CMAKE_CXX_STANDARD_REQUIRED}
#define cmake_cxx_standard_required ${CMAKE_CXX_STANDARD_REQUIRED}
```

在`tutorial.cxx`中引入头文件和使用:

```c++
// TODO 11: Include TutorialConfig.h
#include "TutorialConfig.h"

int main(int argc, char* argv[])
{
  if (argc < 2) {
    // TODO 12: Create a print statement using Tutorial_VERSION_MAJOR
    //          and Tutorial_VERSION_MINOR
    std::cout << Tutorial_VERSION_MAJOR << "." << Tutorial_VERSION_MINOR << std::endl;
    std::cout << "Usage: " << argv[0] << " number" << std::endl;
    return 1;
  }

```

得到的`TutorialConfig.h`如下:

```cpp
// the configured options and settings for Tutorial
// TODO 10: Define Tutorial_VERSION_MAJOR and Tutorial_VERSION_MINOR
#define Tutorial_VERSION_MAJOR 1
#define Tutorial_VERSION_MINOR  1
#define CMAKE_CXX_STANDARD 11
/* #undef CMAKE_CXX_STANDARD_REQUIRED */
#define cmake_cxx_standard_required False
```



## Step 2: Adding a Library

### Exercise 1: Creating a Library

在CMake中使用`add_library`命令可以哪些源文件应该使用库. 使用多个子目录来组织项目.

为我们的库创建一个子目录,为多个源文件添加新的`CMakeLists.txt`文件.在顶层的`CMakeLists.txt`文件中,使用`add_subdirectory`来添加用于构建项目的子目录.

在库被创建之后, 通过`target_include_directories`或者`target_link_libraries`命令和目标链接在一起.

例子: 添加和使用一个库

平方根程序使用自定义实现的求平方根程序而不是标准库实现的.

1.   在`MathFunctions`目录下使用`add_library`创建一个名为`MathFunctions`的库,这个库对应的源代码就是`mysqrt.cxx`:

     ```bash
     # TODO 1: Add a library called MathFunctions
     # Hint: You will need the add_library command
     add_library(MathFunctions  mysqrt.cxx )
     ```

2.   在顶部的`CMakeLists.txt`文件中添加子目录:

     ```bash
     # TODO 2: Use add_subdirectory() to add MathFunctions to this project
     add_subdirectory(MathFunctions)
     ```

     这样在通过顶部的`CMakeLists`构建时,可以调用子目录的配置文件, 构建对应的库.

3.   通过`target_link_libraries`命令将库和目标程序链接到一起:

     ```bash
     # TODO 3: Use target_link_libraries to link the library to our executable
     target_link_libraries(Tutorial PUBLIC MathFunctions)
     ```

4.   使用`target_include_directories`指明库的头文件所在的位置`MathFunctions`子目录, 以便`MathFunctions`头文件可以找到(当时这个头文件的路径写错了,导致编译链接的时候总是找不到头文件)

     ```bash
     # TODO 4: Add MathFunctions to Tutorial's target_include_directories()
     # Hint: ${PROJECT_SOURCE_DIR} is a path to the project source. AKA This folder!
     target_include_directories(Tutorial PUBLIC
              "${PROJECT_BINARY_DIR}"
             "${PROJECT_SOURCE_DIR}/MathFunctions")
     ```

  

 5. 最后, 在源文件中包含头文件并使用对应的函数

     ```cpp
     // TODO 5: Include MathFunctions.h
     #include "MathFunctions.h"
     
       // TODO 6: Replace sqrt with mysqrt
     
       // calculate square root
       const double outputValue = mysqrt(inputValue);
       std::cout << "The square root of " << inputValue << " is " << outputValue
                 << std::endl;
     ```
     



### Exercise 2: Making Our Library Optional

使用`option`来配置一些可选的库

使用`option`来定义一个变量`USE_MYMATH`, 来配置是否使用自定义实现的平方根函数.

1.   在顶部的`CMakeLists.txt`中使用`option`来定义这个变量

     ```bash
     # TODO 7: Create a variable USE_MYMATH using option and set default to ON
     option(USE_MYMATH "use mymath" ON)
     ```

2.   设置在一定条件下才链接`MathFunctions`库.

     使用`list`来创建项目需要使用的可选的库和对应的头文件的链接

     使用`if`来指明条件

     ```bash
     # TODO 8: Use list() and APPEND to create a list of optional libraries
     # called  EXTRA_LIBS and a list of optional include directories called
     # EXTRA_INCLUDES. Add the MathFunctions library and source directory to
     # the appropriate lists.
     #
     # Only call add_subdirectory and only add MathFunctions specific values
     # to EXTRA_LIBS and EXTRA_INCLUDES if USE_MYMATH is true.
     # TODO 2: Use add_subdirectory() to add MathFunctions to this project
     if ( USE_MYMATH )
         add_subdirectory(MathFunctions)
         list(APPEND EXTRA_LIBS MathFunctions)
         list(APPEND EXTRA_INCLUDES "${PROJECT_SOURCE_DIR}/MathFunctions")
     endif()
     ```

3.   使用上面的变量来更新项目需要链接的库和目录

     ```bash
     # TODO 9: Use EXTRA_LIBS instead of the MathFunctions specific values
     # in target_link_libraries.
     target_link_libraries(Tutorial PUBLIC ${EXTRA_LIBS})
     # TODO 10: Use EXTRA_INCLUDES instead of the MathFunctions specific values
     # in target_include_directories.
     target_include_directories(Tutorial PUBLIC
             "${EXTRA_INCLUDES}")
     ```

4.   在源文件中通过`if`条件预编译指令来引入头文件和在条件满足时执行的指令:

     ```c++
     // TODO 11: Only include MathFunctions if USE_MYMATH is defined
     #ifdef USE_MYMATH
     #include "MathFunctions.h"
     #endif
     
       // TODO 12: Use mysqrt if USE_MYMATH is defined and sqrt otherwise
       #ifdef USE_MYMATH
         const double outputValue = mysqrt(inputValue);
       #else
         const double outputValue = sqrt(inputValue);
       #endif
     ```

5.   源代码中使用的`USE_MYMATH`是由`TutorialConfig.h`头文件定义的, 而这个头文件是通过`configure_file`从`TutorialConfig.h.in`头文件复制的, 从而还需要在`TutorialConfig.h.in`中定义这个变量

     ```bash
     // TODO 13: use cmakedefine to define USE_MYMATH
     #cmakedefine USE_MYMATH 
     ```

6.   注: 在通过`option`定义`USE_MYMATH`之后才配置`TutorialConfig.h.in`, 是否可以调换两条语句的位置呢?

     ```bash
     # TODO 7: Create a variable USE_MYMATH using option and set default to ON
     option(USE_MYMATH "use mymath" ON)
     # configure a header file to pass some of the CMake settings
     # to the source code
     configure_file(TutorialConfig.h.in TutorialConfig.h)
     ```

     不可以, 因为在`TutorialConfig.h`中使用了`option`定义的`USE_MYMATH`的值, 如果先配置文件的话, 将没法使用这个`USE_MYMATH`的值,将使用默认的值`OFF`.

     此时查看`TutorialConfig.h`头文件,将是如下结果:

     ```bash
      // TODO 13: use cmakedefine to define USE_MYMATH
      /* #undef USE_MYMATH */
     ```

     

## Step 3: Adding Usage Requirements for a Library

### Exercise 1 - Adding Usage Requirements for a Library

在Step 2的例子中, 我们在顶层的`CMakeLists.txt`文件中包含使用的库`MathFunctions`所在的文件目录, 现在我们通过在`MathFunctions`所在目录的`CMakeLists.txt`文件中显示指明`MathFunctions`所在的目录, 这样在顶层的配置文件中就不用显式包含使用的库的目录.

1.   在`MathFunctions`目录下的配置文件使用使用`target_include_directories`和关键词`INTERFACE`来添加库的依赖.

     ```bash
     add_library(MathFunctions mysqrt.cxx)
     
     # TODO 1: State that anybody linking to MathFunctions needs to include the
     # current source directory, while MathFunctions itself doesn't.
     # Hint: Use target_include_directories with the INTERFACE keyword
     target_include_directories(MathFunctions
             INTERFACE ${CMAKE_CURRENT_SOURCE_DIR}
             )
     ```

2.   从顶层的配置文件移除库相关的目录

     ```bash
     # TODO 2: Remove EXTRA_INCLUDES list
     
     # add the MathFunctions library
     if(USE_MYMATH)
       add_subdirectory(MathFunctions)
       list(APPEND EXTRA_LIBS MathFunctions)
     #  list(APPEND EXTRA_INCLUDES "${PROJECT_SOURCE_DIR}/MathFunctions")
     endif()
     
     target_link_libraries(Tutorial PUBLIC ${EXTRA_LIBS})
     
     # TODO 3: Remove use of EXTRA_INCLUDES
     
     # add the binary tree to the search path for include files
     # so that we will find TutorialConfig.h
     target_include_directories(Tutorial PUBLIC
                                "${PROJECT_BINARY_DIR}"
     #                           ${EXTRA_INCLUDES}
                                )
     
     ```

     这样项目的目标文件只需要使用依赖的库的名称就可以调用库, 不用再像之前那样手动指定库所在的目录.

     

     ## Step 4:  Adding Generator Expressions

     生成表达式在构建项目的时候计算以便产生解释每个构建配置的信息.

     生成表达式Generator expression可以在很多目标特性上使用, 比如`LINK_LIBRARIES`, `INCLUDE_DIRECTORIES`, `COMPILE_DEFINITIONS`等, 也可以通过一些命令使用, 比如`target_link_libraries()`, `target_include_directories`, `target_compile_defineitions`等.

     也可以用于条件链接, 编译时的条件定义, 条件包含目录等.条件可以基于构建项目时的目标特性平台信息等.

     生成表达式包括逻辑, 信息和输出表达式.`$<0:...>`表示一个空串,`$<1:...>`表示字符串`...`

     ### Exercise 1 - Setting the C++ Standard with Interface Libraries

     给`INTERFACE`库目标添加一个指定的C++标准.

     