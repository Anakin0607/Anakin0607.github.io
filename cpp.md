# cmake
# makefile

## 常见预定义变量
|命令格式|含义|
|------|------|
|AR|库文件维护程序的名称，默认值为ar|
|AS|汇编程序的名称，默认值为as|
|CC|C编译器的名称，默认值为gcc|
|CPP|C预编译器(C preprocessor)的名称，默认值为$(CC)-E|
|CXX|C++编译器的名称，默认值为g++|
|FC|FORTRAN编译器的名称，默认值为f77|
|RM|文件删除程序的名称，默认值为rm -f|
|ARFLAGS|库文件维护程序的名称，无默认值|、
|ASFLAGS|汇编程序的选项，无默认值|
|CFLAGS|C编译器的选项，无默认值|
|CPPFLAGS|C预编译器的选项，无默认值，这个选项可以用于c和c++两者|
|CXXFLAGS|C++编译器的选项，无默认值|
|FFLAGS|FORTRAN编译器的选项，无默认值|
|LDFLAGS|用来指定库文件的位置|
|LIBS|告诉链接器需要链接那些库文件|

ar是一个程序，可通过从文档中增加、删除和析取文件来维护库文件。通常使用该工具是为了创建和管理连接程序使用的目标库文档。

as是汇编器，用来编译gcc输出的汇编文件，将汇编转化为二进制代码，并存放到一个目标文件中