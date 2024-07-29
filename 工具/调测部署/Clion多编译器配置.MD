# Clion编译器配置

    Clion、Visit studio、C++、CMake、Debug

## 背景

学习c++，接触到更深一层的内存对齐知识，需要记录，发现有些人说法不一致，遂搭建环境实际测一下，在此简单记一下编译器的配置流程。

## 解决方案

clion安装以后自带mingw编译器，如果要全平台测试，起码要再加个vs编译器：

1. 安装visual studio
2. 打开Clion》file》settings》build，exception，deployment》toolchains
3. 点击左上方“+”，选择visual studio
4. 在toolset后点击browse按钮（文件夹图标的，鼠标放上有提示）
5. 选择到vs安装路径的`..\Microsoft Visual Studio\2022\Community`一级，可以看到下方都自动识别出了信息，可以看到绿色对勾
6. 点击apply
7. 点击`toolchains`下面紧挨的`cmake`菜单
8. 点击左上方“+”
9. `build type`选择`debug`
10. `toolchain`选择`visual studio`，点击`apply`按钮
11. 找到`Debug-Visual Studio`上方的`Debug`，点击选中
12. `toolchain`选择`MinGW`
13. 点击`apply`按钮，确定
14. 把运行按钮旁边的`选择 run/debug 设置`，点击几次
15. 可以看到在这个菜单的运行设置的上面，有两个debug选项，选择一个，然后就可使用对应编译器了
