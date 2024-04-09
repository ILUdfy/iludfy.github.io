---
title: 支持cuda的opencv编译流程
tags:
  - cuda
  - opencv
categories: opencv
date: 2023-04-21 17:12:52
---

# 支持cuda的opencv编译流程
本文中所需文件已上传百度网盘，包括opencv压缩包和编译所需文件，链接如下：
<https://pan.baidu.com/s/1ldnmpgnLIU63MDLOajUp6w>
提取码：bjb6
## 操作环境
1. 显卡：**NVIDIA GeForce RTX 3060**
2. CUDA版本：**CUDA 11.1**
3. 操作系统： **centos7**

## cuda和cudnn安装请看我的另一篇博客
### 温馨提示，在编译支持cuda的opencv之前，要先安装好cuda和cudnn，不然会报错的呢~

## 一、下载源文件
opencv源文件官网链接如下，选择Sources版本下载即可，网盘中附带4.4.0版本：
[Opencv官方下载地址](https://opencv.org/releases/)
下载Contrib对应版本：
[Opencv_Contrib下载地址](https://github.com/opencv/opencv_contrib/releases)
## 二、解压
下载到的是zip格式的压缩包，使用unzip命令解压即可，解压到你想的位置，本文中一用户主目录(/home/dfy)为例：
```bash
unzip opencv-4.4.0.zip
unzip opencv_contrib-4.4.0.zip
```

## 三、安装需要的依赖

### centos系统
组内的服务器使用的是centos7系统，需要安装的依赖如下：
```bash
sudo yum -y install epel-release
sudo yum -y install git gcc gcc-c++ cmake3 # 看到这里的gcc了吗，没错，这里隐藏着一个陨石坑，在后续安装记录章节细说
sudo yum -y install qt5-qtbase-devel
sudo yum install -y python34 python34-devel python34-pip
sudo yum install -y python python-devel python-pip

sudo yum -y install python-devel numpy python34-numpy
sudo yum -y install gtk2-devel

sudo yum install -y libpng-devel
sudo yum install -y jasper-devel
sudo yum install -y openexr-devel
sudo yum install -y libwebp-devel
sudo yum -y install libjpeg-turbo-devel 
sudo yum install -y freeglut-devel mesa-libGL mesa-libGL-devel
sudo yum -y install libtiff-devel 
sudo yum -y install libdc1394-devel
sudo yum -y install tbb-devel eigen3-devel
sudo yum -y install boost boost-thread boost-devel
sudo yum -y install libv4l-devel
sudo yum -y install gstreamer-plugins-base-devel
```
安装编译所需要的cmake3(centos7系统有点老了，默认的cmake版本不够，需要指定cmake3)
```bash
sudo yum install epel-release
sudo yum install cmake3
```

安装tesseract
```bash
# 搜索「tesseract」
yum search tesseract

#安装「tesseract.x86_64」
yum install tesseract.x86_64 tesseract-devel.x86_64 

#安装「tesseract-langpack-chi_sim.noarch」中文字库
yum install tesseract-langpack-chi_sim.noarch

#检查「tesseract」支持的语言
tesseract --list-langs
```

### ubuntu系统
```
$ sudo apt-get update
$ sudo apt-get upgrade
$ sudo apt-get install build-essential cmake unzip pkg-config
$ sudo apt-get install libjpeg-dev libpng-dev libtiff-dev
$ sudo apt-get install libavcodec-dev libavformat-dev libswscale-dev
$ sudo apt-get install libv4l-dev libxvidcore-dev libx264-dev
$ sudo apt-get install libgtk-3-dev
$ sudo apt-get install libatlas-base-dev gfortran
$ sudo apt-get install python3-dev
```

### 安装numpy
```
# 执行这一步之前，激活你要使用的python虚拟环境
pip install numpy
```

## 四、执行cmake
```bash
# 进入opencv的解压目录
cd ~/opencv-4.4.0

# 创建构建目录build（这里build目录名称随意）
mkdir build
cd build

# 执行cmake
# 这里一些参数的配置一定要仔细再仔细
# 我的参数配置如下
# centos系统如果提示cmake版本不够，这里输入cmake3
cmake -D CMAKE_BUILD_TYPE=RELEASE \
        -D CMAKE_INSTALL_PREFIX=/usr/local \
        -D INSTALL_PYTHON_EXAMPLES=ON \
        -D INSTALL_C_EXAMPLES=OFF \
        -D OPENCV_ENABLE_NONFREE=ON \
        -D WITH_CUDA=ON \
        -D WITH_CUDNN=ON \
        -D OPENCV_DNN_CUDA=ON \
        -D ENABLE_FAST_MATH=1 \
        -D CUDA_FAST_MATH=1 \
        -D CUDA_ARCH_BIN=8.6 \
        -D WITH_CUBLAS=1 \
        -D OPENCV_EXTRA_MODULES_PATH=~/opencv_contrib-4.4.0/modules \
        -D HAVE_opencv_python3=ON \
        -D PYTHON_EXECUTABLE=/usr/local/bin/python3 \
        -D BUILD_EXAMPLES=ON ..
       ..
```
详细解释一下命令中各个参数选取的注意事项：
> 1. OPENCV_EXTRA_MODULES_PATH，翻译过来就是opencv额外模块路径，这个模块就是我们下载的Contrib模块。所以这项参数填写你的opencv_contrib目录下modules文件夹的路径。如果你是按照上面的教程走的，那么它现在应该在你的用户主目录下：~/opencv_contrib-4.4.0/modules。
> 3. PYTHON_EXECUTABLE，python虚拟环境路径，它需要指向你的环境中python的可执行文件。若是anaconda的虚拟环境，它一般在虚拟环境目录下的bin目录下：~/anaconda3/envs/mouse/bin/python3.9,或者是系统的python3：/usr/local/bin/python3(不建议，我虽然是这么做的，但是我是在docker镜像内操作，系统本身的python关系到系统的运行，比较危险，如果实在要用，可以安装一下其他版本的python，用其他版本来操作)
>    python版本可能很多，指定你所需版本的python可执行文件。
> 4. CUDA_ARCH_BIN，这里指的是你的cuda版本对应的算力级别。去[cuda官网](https://developer.nvidia.com/cuda-gpus#collapseOne)查看一下。组内服务器的显卡是3090，cuda版本我装的是11.1，官网上写着对应的算力级别为8.6，所以这里填8.6。
> 5. 命令最后两个点不能落下，这两个点代表相对路径。

以上是实际应用中需要自己确定的问题，其他的按照我写的命令复制即可。
我在执行cmake的时候报了一些错误，很多个failed，但是实际使用并没什么问题。
如果你执行命令之后，发现你的cmake参数配置有错误，那么你需要删除build目录重新执行这一步骤。

**在执行下一步之前，你需要检查cmake的输出，有个至关重要的点：**
检查cmake输出中有没有这个模块：
```
--   Python 3:
--     Interpreter:                 /usr/local/bin/python3 (ver 3.9.16)
--     Libraries:                   /usr/local/lib/libpython3.9.so (ver 3.9.16)
--     numpy:                       /usr/local/lib/python3.9/site-packages/numpy/core/include (ver 1.24.2)
--     install path:                lib/python3.9/site-packages/cv2/python-3.9
```
如果没有这一块，没必要往下走了，因为安装完也没有用。你需要检查你的python环境是否具有python3的条件。
这里输出的```install path```后面要用到，先记下来。

其他的信息你可以检查一下你配置是否正确，例如下面：
```
     NVIDIA CUDA:                   YES (ver 11.1, CUFFT CUBLAS FAST_MATH)
       NVIDIA GPU arch:             86
       NVIDIA PTX archs:
  
     cuDNN:                         YES (ver 8.7.0)
```

## 五、安装
```bash
make -j$(nproc)
```
安装过程中也会踩很多很多很多很多很多的陨石坑，在这里记录一下（本人被折磨的不轻）

### 1.Unsupported gpu architecture 'compute_30'
翻译过来就是gpu算力不支持，说明你在cmake的CUDA_ARCH_BIN设置和你的显卡和cuda不匹配，去[cuda官网](https://developer.nvidia.com/cuda-gpus#collapseOne)找你的显卡对应的算力。

### 2.# error "OpenCV 4.x+ requires enabled C++11 support"
这句话意思是opencv4以上的版本需要c++11才可以编译。
所以这里我去看一下gcc的版本，看看支不支持c++11.
gcc的版本是4.8.5，按理说应该是支持c++11的，但是这里编译就是过不了。
网络上的解决办法是在编译的时候加上```-std=c++11```这个选项。
但是编译语句都是cmake自动生成的，那么多语句，要找到猴年马月。
一度怀疑人生。

后来我仔细看了下报错信息，发现在之前还有这样一句话：
```nvcc warning : The -std=c++14 flag is not supported with the configured host compiler. Flag will be ignored. ```
这句话后面几行才打出```# error "OpenCV 4.x+ requires enabled C++11 support"```这句话，那么说明根源在上面那句。

这句话的意思是，编译选项```-std=c++14```不支持，所以该选项被忽略。
说明编译语句中写的是```-std=c++14```，而不是```-std=c++11```，说明尽管gcc4.8.5支持c++11，它还是没有办法去调用，因为编译语句没有指定。
那么从根源解决问题，升高gcc版本，让它支持c++14，该问题才能得到解决。

所以我把gcc版本升级到了9.3.0，该错误消失了。

真好啊，ctmd，浪费老子半天。

gcc安装教程参考博客<https://blog.csdn.net/zblock0/article/details/107032359>
### 3. fatal error: boostdesc_bgm.i:no such file or directory
说明缺少一些依赖文件。文件在文章开始的百度网盘提供了。把所有的.i文件复制到opencv_contrib/modules/xfeatures2d/src/目录下即可。
### 4. fatal error: features2d/test/test_detectors_regression.impl.hpp:no such file or directory
依旧是缺少文件，这个文件在opencv的modules目录下其实已经提供了，是路径的问题。所以复制一下：
```cp -r ../modules/features2d ./```

所有这种缺少文件的报错，都可以去找一下这个文件的名字，如果在opencv或者contrib中提供了，复制过去就好。

### 5.CMakeFiles/example_gpu_surf_keypoint_matcher.dir/surf_keypoint_matcher.cpp.o: In function `main’

完整报错信息如下：
```
CMakeFiles/example_gpu_surf_keypoint_matcher.dir/surf_keypoint_matcher.cpp.o: In function `main':
surf_keypoint_matcher.cpp:(.text.startup.main+0x352): undefined reference to `cv::cuda::SURF_CUDA::SURF_CUDA()'
surf_keypoint_matcher.cpp:(.text.startup.main+0x579): undefined reference to `cv::cuda::SURF_CUDA::operator()(cv::cuda::GpuMat const&, cv::cuda::GpuMat const&, cv::cuda::GpuMat&, cv::cuda::GpuMat&, bool)'
surf_keypoint_matcher.cpp:(.text.startup.main+0x60d): undefined reference to `cv::cuda::SURF_CUDA::operator()(cv::cuda::GpuMat const&, cv::cuda::GpuMat const&, cv::cuda::GpuMat&, cv::cuda::GpuMat&, bool)'
surf_keypoint_matcher.cpp:(.text.startup.main+0x6af): undefined reference to `cv::cuda::SURF_CUDA::defaultNorm() const'
surf_keypoint_matcher.cpp:(.text.startup.main+0x7ca): undefined reference to `cv::cuda::SURF_CUDA::downloadKeypoints(cv::cuda::GpuMat const&, std::vector<cv::KeyPoint, std::allocator<cv::KeyPoint> >&)'
surf_keypoint_matcher.cpp:(.text.startup.main+0x7ea): undefined reference to `cv::cuda::SURF_CUDA::downloadKeypoints(cv::cuda::GpuMat const&, std::vector<cv::KeyPoint, std::allocator<cv::KeyPoint> >&)'
surf_keypoint_matcher.cpp:(.text.startup.main+0x800): undefined reference to `cv::cuda::SURF_CUDA::downloadDescriptors(cv::cuda::GpuMat const&, std::vector<float, std::allocator<float> >&)'
surf_keypoint_matcher.cpp:(.text.startup.main+0x812): undefined reference to `cv::cuda::SURF_CUDA::downloadDescriptors(cv::cuda::GpuMat const&, std::vector<float, std::allocator<float> >&)'
collect2: error: ld returned 1 exit status
samples/gpu/CMakeFiles/example_gpu_surf_keypoint_matcher.dir/build.make:132: recipe for target 'bin/example_gpu_surf_keypoint_matcher' failed
make[2]: *** [bin/example_gpu_surf_keypoint_matcher] Error 1
```
解决方法：
先去你的build目录下找到这个文件:
```<build_dir>/samples/gpu/CMakeFiles/example_gpu_surf_keypoint_matcher.dir/link.txt```
在这个文件里，搜索到```--as-needed CMakeFiles/example_gpu_surf_keypoint_matcher.dir/surf_keypoint_matcher.cpp.o```这一句之后加上```<build_dir>/modules/xfeatures2d/CMakeFiles/opencv_xfeatures2d.dir/src/surf.cuda.cpp.o <build_dir>/modules/xfeatures2d/CMakeFiles/cuda_compile_1.dir/src/cuda/cuda_compile_1_generated_surf.cu.o```
其中，<build_dir>是你的构建路径，也就是第四步中创建的build文件夹。

### 6.libopencv_imgcodecs.so.4.2.0: undefined reference to `TIFFReadRGBAStrip@LIBTIFF_4.0
完整报错信息：
 ```
/usr/local/lib/libopencv_imgcodecs.so.4.2.0: undefined reference to `TIFFReadRGBAStrip@LIBTIFF_4.0'
/usr/local/lib/libopencv_imgcodecs.so.4.2.0: undefined reference to `TIFFReadDirectory@LIBTIFF_4.0'
/usr/local/lib/libopencv_imgcodecs.so.4.2.0: undefined reference to `TIFFWriteEncodedStrip@LIBTIFF_4.0'
/usr/local/lib/libopencv_imgcodecs.so.4.2.0: undefined reference to `TIFFIsTiled@LIBTIFF_4.0'
/usr/local/lib/libopencv_imgcodecs.so.4.2.0: undefined reference to `TIFFWriteScanline@LIBTIFF_4.0'
/usr/local/lib/libopencv_imgcodecs.so.4.2.0: undefined reference to `TIFFGetField@LIBTIFF_4.0'
/usr/local/lib/libopencv_imgcodecs.so.4.2.0: undefined reference to `TIFFScanlineSize@LIBTIFF_4.0'
/usr/local/lib/libopencv_imgcodecs.so.4.2.0: undefined reference to `TIFFWriteDirectory@LIBTIFF_4.0'
/usr/local/lib/libopencv_imgcodecs.so.4.2.0: undefined reference to `TIFFReadEncodedTile@LIBTIFF_4.0'
/usr/local/lib/libopencv_imgcodecs.so.4.2.0: undefined reference to `TIFFReadRGBATile@LIBTIFF_4.0'
/usr/local/lib/libopencv_imgcodecs.so.4.2.0: undefined reference to `TIFFClose@LIBTIFF_4.0'
/usr/local/lib/libopencv_imgcodecs.so.4.2.0: undefined reference to `TIFFClientOpen@LIBTIFF_4.0'
/usr/local/lib/libopencv_imgcodecs.so.4.2.0: undefined reference to `TIFFRGBAImageOK@LIBTIFF_4.0'
/usr/local/lib/libopencv_imgcodecs.so.4.2.0: undefined reference to `TIFFOpen@LIBTIFF_4.0'
/usr/local/lib/libopencv_imgcodecs.so.4.2.0: undefined reference to `TIFFReadEncodedStrip@LIBTIFF_4.0'
/usr/local/lib/libopencv_imgcodecs.so.4.2.0: undefined reference to `TIFFSetField@LIBTIFF_4.0'
/usr/local/lib/libopencv_imgcodecs.so.4.2.0: undefined reference to `TIFFSetWarningHandler@LIBTIFF_4.0'
/usr/local/lib/libopencv_imgcodecs.so.4.2.0: undefined reference to `TIFFSetErrorHandler@LIBTIFF_4.0
 ```

 库引用的问题，从报错信息中可以看到是```libopencv_imgcodecs.so.4.2.0```这个库引用不到一个名字含有```TIFF```的库。
 首先你要检查依赖：```libtiff-dev```是否正确安装。
 如果正确安装后还是出现这个问题，那么可能是你的python环境下没有这个库，你需要复制一份到你的python环境下。
 具体操作如下：
```
  # 先查看报错信息，发现是```/usr/local/lib/libopencv_imgcodecs.so.4.2.0```库引用不到其他库
  # 查看这个库的依赖
  ldd /usr/local/lib/libopencv_imgcodecs.so.4.2.0

  # 输出如下
  linux-vdso.so.1 =>  (0x00007ffdc29f5000)
        libopencv_imgproc.so.4.2 => /usr/local/lib/libopencv_imgproc.so.4.2 (0x00007fc9c680c000)
        libjpeg.so.8 => /usr/lib/x86_64-linux-gnu/libjpeg.so.8 (0x00007fc9c65b3000)
        libpng12.so.0 => /lib/x86_64-linux-gnu/libpng12.so.0 (0x00007fc9c638e000)
        libtiff.so.5 => /usr/lib/x86_64-linux-gnu/libtiff.so.5 (0x00007fc9c6119000)
        libjasper.so.1 => /usr/lib/x86_64-linux-gnu/libjasper.so.1 (0x00007fc9c5ec4000)
        libIlmImf-2_2.so.22 => /usr/lib/x86_64-linux-gnu/libIlmImf-2_2.so.22 (0x00007fc9c59f5000)
        libopencv_core.so.4.2 => /usr/local/lib/libopencv_core.so.4.2 (0x00007fc9c5318000)
        libstdc++.so.6 => /usr/lib/x86_64-linux-gnu/libstdc++.so.6 (0x00007fc9c4f96000)
        libm.so.6 => /lib/x86_64-linux-gnu/libm.so.6 (0x00007fc9c4c8d000)
        libgcc_s.so.1 => /lib/x86_64-linux-gnu/libgcc_s.so.1 (0x00007fc9c4a77000)
        libpthread.so.0 => /lib/x86_64-linux-gnu/libpthread.so.0 (0x00007fc9c485a000)
        libc.so.6 => /lib/x86_64-linux-gnu/libc.so.6 (0x00007fc9c4490000)
        libz.so.1 => /usr/local/lib/libz.so.1 (0x00007fc9c4272000)
        liblzma.so.5 => /lib/x86_64-linux-gnu/liblzma.so.5 (0x00007fc9c4050000)
        libjbig.so.0 => /usr/lib/x86_64-linux-gnu/libjbig.so.0 (0x00007fc9c3e42000)
        libHalf.so.12 => /usr/lib/x86_64-linux-gnu/libHalf.so.12 (0x00007fc9c3bff000)
        libIex-2_2.so.12 => /usr/lib/x86_64-linux-gnu/libIex-2_2.so.12 (0x00007fc9c39e1000)
        libIlmThread-2_2.so.12 => /usr/lib/x86_64-linux-gnu/libIlmThread-2_2.so.12 (0x00007fc9c37da000)
        libdl.so.2 => /lib/x86_64-linux-gnu/libdl.so.2 (0x00007fc9c35d6000)
        librt.so.1 => /lib/x86_64-linux-gnu/librt.so.1 (0x00007fc9c33ce000)
        libGL.so.1 => /usr/lib/x86_64-linux-gnu/mesa/libGL.so.1 (0x00007fc9c315a000)
        /lib64/ld-linux-x86-64.so.2 (0x00007fc9c73b1000)
        libexpat.so.1 => /lib/x86_64-linux-gnu/libexpat.so.1 (0x00007fc9c2f31000)
        libxcb-dri3.so.0 => /usr/lib/x86_64-linux-gnu/libxcb-dri3.so.0 (0x00007fc9c2d2e000)
        libxcb-present.so.0 => /usr/lib/x86_64-linux-gnu/libxcb-present.so.0 (0x00007fc9c2b2b000)
        libxcb-sync.so.1 => /usr/lib/x86_64-linux-gnu/libxcb-sync.so.1 (0x00007fc9c2924000)
        libxshmfence.so.1 => /usr/lib/x86_64-linux-gnu/libxshmfence.so.1 (0x00007fc9c2721000)
        libglapi.so.0 => /usr/lib/x86_64-linux-gnu/libglapi.so.0 (0x00007fc9c24f0000)
        libXext.so.6 => /usr/lib/x86_64-linux-gnu/libXext.so.6 (0x00007fc9c22de000)
        libXdamage.so.1 => /usr/lib/x86_64-linux-gnu/libXdamage.so.1 (0x00007fc9c20db000)
        libXfixes.so.3 => /usr/lib/x86_64-linux-gnu/libXfixes.so.3 (0x00007fc9c1ed5000)
        libX11-xcb.so.1 => /usr/lib/x86_64-linux-gnu/libX11-xcb.so.1 (0x00007fc9c1cd3000)
        libX11.so.6 => /usr/lib/x86_64-linux-gnu/libX11.so.6 (0x00007fc9c1999000)
        libxcb-glx.so.0 => /usr/lib/x86_64-linux-gnu/libxcb-glx.so.0 (0x00007fc9c1780000)
        libxcb-dri2.so.0 => /usr/lib/x86_64-linux-gnu/libxcb-dri2.so.0 (0x00007fc9c157b000)
        libxcb.so.1 => /usr/lib/x86_64-linux-gnu/libxcb.so.1 (0x00007fc9c1359000)
        libXxf86vm.so.1 => /usr/lib/x86_64-linux-gnu/libXxf86vm.so.1 (0x00007fc9c1153000)
        libdrm.so.2 => /usr/lib/x86_64-linux-gnu/libdrm.so.2 (0x00007fc9c0f41000)
        libXau.so.6 => /usr/lib/x86_64-linux-gnu/libXau.so.6 (0x00007fc9c0d3d000)
        libXdmcp.so.6 => /usr/lib/x86_64-linux-gnu/libXdmcp.so.6 (0x00007fc9c0b37000)
  
  # 你需要在这些依赖中找到报错信息中找不到的那个库。
  # 这个库在这里：
  libtiff.so.5 => /usr/lib/x86_64-linux-gnu/libtiff.so.5 (0x00007fc9c6119000)

  # 把这个库复制到你的python环境下：
  sudo cp /usr/lib/x86_64-linux-gnu/libtiff.so.5 /usr/local/lib/libtiff.so.5
```

不出意外的话应该是没什么其他的问题了。

make完成后，执行：
```bash
sudo make install
 ```

## 六、链接动态库
安装完成后，在之前cmake输出的python3 ```install path```路径下，会出现一个库：
我之前这里输出的```install path = lib/python3.9/site-packages/cv2/python-3.9```，在这个路径前加上安装前缀```CMAKE_INSTALL_PREFIX=/usr/local```
```
ls -l /usr/local/lib/python3.9/site-packages/cv2/python-3.9


总计 9540
-rw-r--r-- 1 root root 9768648  5月  4 17:51 cv2.cpython-39-x86_64-linux-gnu.so

```
把这个库连接到你的python环境下：
```
cd /usr/local/lib/python3.9/site-packages #这里要看你自己的python环境路径，我是使用的系统的python，所以库直接生成在了我需要的python环境下，这一步对我其实可以省略
ln -s /usr/local/lib/python3.9/site-packages/cv2/python3.9/cv2.cpython-39-x86_64-linux-gnu.so cv2.so
```


## 测试
```bash
#激活你安装了opencv的虚拟环境，如果你用的系统python，这一步不需要
conda activate mouse

#打开python
python

#测试
import cv2
cv2.__version__
# 输出：'4.4.0'
```