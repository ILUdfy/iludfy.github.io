---
title: matlab编译dll文件
date: 2023-03-14 13:18:39
tags: matlab
---
# matlab编译dll文件
1.命令行窗口输入：```deploytool```，会出现如下弹窗：
![img](./deploytool.png)


选择第三个，出现下面的窗口，按图示操作：

![img](./deploytool2.png)

打包好后会出现三个文件夹：

![img](./deploytool3.png)

1. for_redistribution文件夹下存放一个exe文件，它用来安装使用dll库所需要的matlab环境，一般在没有matlab的机器上，可以使用该文件快捷安装所需环境；
2. for_redistribution_files_only文件夹下就是编译生成的库文件、头文件和dll文件了