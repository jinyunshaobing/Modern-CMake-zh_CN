# 与代码通讯

## 配置文件（Configure File）

CMake允许使用`configuration_file`命令从代码中访问CMake变量。这个命令会将文件（通常以`.in`结尾）从一个位置拷贝至另一个位置，并替换它找到的CMake变量。如果你想避免替换输入文件现有的 `${}`语法，请使用`@ONLY`关键字。如果你只是用它来代替`file(COPY`命令，可以使用`COPY_ONLY` 关键字。

这个功能使用频率很高。例如，在 `Version.h.in`中

#### Version.h.in

```cpp
#pragma once

#define MY_VERSION_MAJOR @PROJECT_VERSION_MAJOR@
#define MY_VERSION_MINOR @PROJECT_VERSION_MINOR@
#define MY_VERSION_PATCH @PROJECT_VERSION_PATCH@
#define MY_VERSION_TWEAK @PROJECT_VERSION_TWEAK@
#define MY_VERSION "@PROJECT_VERSION@"
```

#### CMake lines:
```cmake
configure_file (
    "${PROJECT_SOURCE_DIR}/include/My/Version.h.in"
    "${PROJECT_BINARY_DIR}/include/My/Version.h"
)
```

**在构建项目时，您还应该包括二进制包含目录**。如果想在头部设置true/false变量，CMake提供了C中对应`#cmakedefine` 和 `#cmakedefine01`的特定替换来生成适当的预处理行。

You should include the binary include directory as well when building your project. If you want to put any true/false variables in a header, CMake has C specific `#cmakedefine` and `#cmakedefine01` replacements to make appropriate define lines.

您也可以（并且经常）使用它来生成`.cmake`文件，例如配置文件
You can also (and often do) use this to produce `.cmake` files, **such as the configure files (see [installing](https://cliutils.gitlab.io/modern-cmake/chapters/install/installing.html))**.

## 读取文件

The other direction can be done too; you can read in something (like a version) from your source files. **If you have a header only library that you'd like to make available with or without CMake,** for example, then this would be the best way to handle a version. This would look something like this:

还有另外一种和代码通讯的方式。您可以从源文件中读取某些内容（例如版本号）。例如，如果你想要在使用或不使用CMake的情况下连接一个仅包含头文件的库，那么读取文件将是最好的处理版本的方式。做法如下：

```cmake
# Assuming the canonical version is listed in a single line
# This would be in several parts if picking up from MAJOR, MINOR, etc.
set(VERSION_REGEX "#define MY_VERSION[ \t]+\"(.+)\"")

# Read in the line containing the version
file(STRINGS "${CMAKE_CURRENT_SOURCE_DIR}/include/My/Version.hpp"
    VERSION_STRING REGEX ${VERSION_REGEX})

# Pick out just the version
string(REGEX REPLACE ${VERSION_REGEX} "\\1" VERSION_STRING "${VERSION_STRING}")

# Automatically getting PROJECT_VERSION_MAJOR, My_VERSION_MAJOR, etc.
project(My LANGUAGES CXX VERSION ${VERSION_STRING})
```

Above, `file(STRINGS file_name variable_name REGEX regex)` picks lines that match a regex; and the same regex is used to then pick out the parentheses capture group with the version part.**Replace is used with back substitution to output only that one group.**

如上， `file(STRINGS file_name variable_name REGEX regex)`在文件中选出匹配正则表达式的行，然后用同样的正则表达式选出带有版本信息的括号捕获组。