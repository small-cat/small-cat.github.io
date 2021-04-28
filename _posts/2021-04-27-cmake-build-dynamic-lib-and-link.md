---
layout: post
title: "cmake 编译带有版本号的动态库和链接不带版本的动态库"
date: 2021-04-27 23:00:00
tags: cmake c++ linux
---
cmake 中通过 `add_library` 可以编译带有版本号的动态库，但是链接这个动态库时，如何指定不带有版本号的动态库名称呢，本文带你找到答案

# cmake 编译带有版本号的动态库和链接不带版本的动态库

cmake 中，通过 `add_library` 的方式，来设置编译目标，编译结果为动态库或者静态库

```
add_library(<name> [STATIC | SHARED | MODULE]
            [EXCLUDE_FROM_ALL]
            [source1] [source2 ...])
```

name 就是目标名，即 `target_name`。目标名称，在 cmake 中是一个很特殊的存在，特殊在哪里呢，后面我一点点展开说明。

上面的参数中，STATIC 表示目标为静态库，而 SHARED 表示为动态库。

我们来看一个例子，例子很简单，就是实现一个 output 打印接口，编译成动态库 `liboutput.so`，然后通过链接这个动态库的方式调用 output 方法，打印 Hello World 到屏幕上，我们来看一下目录结构

```
├── CMakeLists.txt
├── demo
│   ├── CMakeLists.txt
│   └── helloworld.cpp
├── output.cpp
├── output.h
```

根目录中的 `CMakeLists.txt` 文件为

```
project(test)

add_library(output SHARED output.cpp)
set(LIB_OUTPUT_DIR "${PROJECT_SOURCE_DIR}/dist")
set_target_properties(output
  PROPERTIES
  LIBRARY_OUTPUT_DIRECTORY ${LIB_OUTPUT_DIR}
  ARCHIVE_OUTPUT_DIRECTORY ${LIB_OUTPUT_DIR}
  )

add_subdirectory(demo)
```

为了方便，我们通过 `set_target_properties` 将动态库编译后，存放到根目录下的 dist 文件夹中，`${PROJECT_SOURCE_DIR}` 这个变量所代表的目录，跟 project 有关，表示的是指定了 project 的目录作为源代码路径，也就是 `${PROJECT_SOURCE_DIR}` 这个变量的值。

而 `demo/CMakeLists.txt` 为

```
cmake_minimum_required(VERSION 2.8)

include_directories(${PROJECT_SOURCE_DIR})
add_executable(helloworld helloworld.cpp)
target_link_libraries(helloworld PUBLIC output)
```

编译成可执行文件 helloworld，编译成功后，看下链接的情况

```
 # ldd helloworld 
	linux-vdso.so.1 (0x00007ffc93ff9000)
	liboutput.so => /home/jona/test/dist/liboutput.so (0x00007f1ba8c16000)
	libstdc++.so.6 => /usr/lib/x86_64-linux-gnu/libstdc++.so.6 (0x00007f1ba888d000)
	libgcc_s.so.1 => /lib/x86_64-linux-gnu/libgcc_s.so.1 (0x00007f1ba8675000)
	libc.so.6 => /lib/x86_64-linux-gnu/libc.so.6 (0x00007f1ba8284000)
	libm.so.6 => /lib/x86_64-linux-gnu/libm.so.6 (0x00007f1ba7ee6000)
	/lib64/ld-linux-x86-64.so.2 (0x00007f1ba901b000)

```

在 dist 目录下的动态库也编译成功了，看下目录情况

```
├── CMakeLists.txt
├── demo
│   ├── CMakeLists.txt
│   └── helloworld.cpp
├── dist
│   ├── liboutput.so
├── output.cpp
├── output.h
```

可执行程序 helloworld 已经成功链接到了 `liboutput.so` 这个动态库上。

一般来说，我们编译动态库的时候，都会加上版本号，比如 `liboutput.so.0.0.1` ，然后可执行文件在链接的时候，链接到 `liboutput.so`，让 `liboutput.so` 是 `liboutput.so.0.0.1` 的软链接即可。这样，我们在升级不同版本的动态库的时候，只需要修改软链接执行不同版本的动态库即可，不需要重新编译链接源程序。

但是，笔者在使用 cmake 的时候，就遇到了一些坑。我通过下面这种方式来编译的动态库，根目录下的 `CMakeLists.txt` 改为

```
project(test)

add_library(output SHARED output.cpp)

file(STRINGS "VERSION" LIB_VERSION)

set(LIB_OUTPUT_DIR "${PROJECT_SOURCE_DIR}/dist")

set_target_properties(output
  PROPERTIES
  VERSION ${LIB_VERSION}
  SOVERSION ${LIB_VERSION}
  LIBRARY_OUTPUT_DIRECTORY ${LIB_OUTPUT_DIR}
  ARCHIVE_OUTPUT_DIRECTORY ${LIB_OUTPUT_DIR}
  )

add_subdirectory(demo)
```

在 `set_target_properties` 中，加上了 SOVERSION 版本号，这样，在编译的时候，就会编译成带有版本号的动态库文件，然后创建一个不带有版本号的软链接，变成完成的库如下所示

```
lrwxrwxrwx 1 jona jona   18 Apr 23 18:19 liboutput.so -> liboutput.so.0.0.1
-rwxrwxr-x 1 jona jona 8648 Apr 23 18:19 liboutput.so.0.0.1
```

再看下可在执行文件的情况

```
# ldd helloworld 
	linux-vdso.so.1 (0x00007ffdedbfc000)
	liboutput.so.0.0.1 => /home/jona/test/dist/liboutput.so.0.0.1 (0x00007f5e480d2000)
	libstdc++.so.6 => /usr/lib/x86_64-linux-gnu/libstdc++.so.6 (0x00007f5e47d49000)
	libgcc_s.so.1 => /lib/x86_64-linux-gnu/libgcc_s.so.1 (0x00007f5e47b31000)
	libc.so.6 => /lib/x86_64-linux-gnu/libc.so.6 (0x00007f5e47740000)
	libm.so.6 => /lib/x86_64-linux-gnu/libm.so.6 (0x00007f5e473a2000)
	/lib64/ld-linux-x86-64.so.2 (0x00007f5e484d7000)
```

链接的是 `liboutput.so.0.0.1` ，为什么不是链接的 `liboutput.so` 呢，虽然我们在 `CMakeLists.txt` 文件中是

```
target_link_libraries(helloworld PUBLIC output)
```

这样指定的动态库。这里的 output 是目标名，也就是上面我们通过

 ```
add_library(output SHARED output.cpp)
 ```

指定的目标名，cmake 在链接的时候，通过这个 target name 找到这个库的完整路径进行链接，而库的名称就是`liboutput.so.0.0.1`，而不是 `liboutput.so`。我们可以看一下 cmake 生成的链接文件 `link.txt`

```
/usr/bin/c++    -rdynamic CMakeFiles/helloworld.dir/helloworld.cpp.o  -o helloworld -Wl,-rpath,/home/jona/Documents/programming/c_plus_plus/cmakefile_test/test/dist ../../dist/liboutput.so.0.0.1 
```

链接的时候，直接使用的就是带有版本号的库文件名。

注意，这里即使按照下面这种方式进行链接

```
/usr/bin/c++    -rdynamic CMakeFiles/helloworld.dir/helloworld.cpp.o  -o helloworld -Wl,-rpath,/home/jona/Documents/programming/c_plus_plus/cmakefile_test/test/dist ../../dist/liboutput.so
```

用 ldd 查看结果，仍然链接的是带版本好的库，因为不带版本号的库，就是一个软链接，实际的库就是带有版本号的。

> target_link_libraries 中指定链接库的方式有如下这几种
>
> - **A library target name**，就是上面我们使用到的
> - **A full path to a library file**，这是指定库的完整路径的方式
> - **A plain library name**，这种方式比较特殊，cmake 会将这种方式翻译成 `-lname` 或者 `name.lib` 的方式
>
> 比如，我们将上面的改成 `target_link_libraries(helloworld PUBLIC output.so)` 的方式，`link.txt` 就变成了
>
> ```
> /usr/bin/c++    -rdynamic CMakeFiles/helloworld.dir/helloworld.cpp.o  -o helloworld -Wl,-rpath,/home/jona/Documents/programming/c_plus_plus/cmakefile_test/test/dist -loutput
> ```
>
> - **A link flag**，这种方式，在名称前面加上 `-`，就变成了 linker 的选项了

那么，我们如何才能做到预期的那样，直接链接到不带版本号的库呢，借助一点小技巧。在编译成动态库的时候，不加版本号，在编译结束后，将库重命名成带有版本号的库，然后创建库的软链接为不带版本号的库，`CMakeLists.txt` 文件改成如下的方式

```
project(test)

add_library(output SHARED output.cpp)

file(STRINGS "VERSION" LIB_VERSION)

set(LIB_OUTPUT_DIR "${PROJECT_SOURCE_DIR}/dist")

set_target_properties(output
  PROPERTIES
  LIBRARY_OUTPUT_DIRECTORY ${LIB_OUTPUT_DIR}
  ARCHIVE_OUTPUT_DIRECTORY ${LIB_OUTPUT_DIR}
  )

add_custom_command(TARGET output POST_BUILD
  COMMAND
  mv liboutput.so liboutput.so.${LIB_VERSION}
  COMMAND
  ln -s liboutput.so.${LIB_VERSION} liboutput.so
  WORKING_DIRECTORY ${LIB_OUTPUT_DIR}
  )

add_subdirectory(demo)
```

再看下可执行文件的链接情况

```
# ldd helloworld 
	linux-vdso.so.1 (0x00007fff453c9000)
	liboutput.so => /home/jona/test/dist/liboutput.so (0x00007f6209f38000)
	libstdc++.so.6 => /usr/lib/x86_64-linux-gnu/libstdc++.so.6 (0x00007f6209baf000)
	libgcc_s.so.1 => /lib/x86_64-linux-gnu/libgcc_s.so.1 (0x00007f6209997000)
	libc.so.6 => /lib/x86_64-linux-gnu/libc.so.6 (0x00007f62095a6000)
	libm.so.6 => /lib/x86_64-linux-gnu/libm.so.6 (0x00007f6209208000)
	/lib64/ld-linux-x86-64.so.2 (0x00007f620a33d000)

```

这样就达到我们的预期了
