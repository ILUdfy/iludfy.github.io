---
title: centos7下配置ssd和yolov5的环境
date: 2023-03-08 15:22:40
tags: 
    - pytorch
    - ssd
    - yolov5
    - centos
categories: pytorch
---
# centos下配置ssd和yolov5的环境
## 1.安装anaconda
首先确定anaconda的版本，我这里用得是python3.9，查到对应的anaconda版本为Anaconda3-2021.11<br>
**在用户主目录下：**
```bash
wget https://repo.anaconda.com/archive/Anaconda3-2021.11-Linux-x86_64.sh
```
之后用户主目录下回多出一个Anaconda3-2021.11-Linux-x86_64.sh文件，直接运行即可：
```bash
./Anaconda3-2021.11-Linux-x86_64.sh
```
会强制看完用户协议，一直按回车即可。
最后会问：
```
Do you accept the license terms? [yes|no][no] 
>>> yes
```
输入yes即可。
之后会让你确定安装路径：
```
# 使用默认路径，直接键入回车，使用自定义路径，直接输入安装路径
# 此处使用默认路径作为安装路径
Anaconda3 will now be installed into this location:
/home/dfy/anaconda3  
- Press ENTER to confirm the location  
- Press CTRL-C to abort the installation  
- Or specify a different location below
 
[/home/dfy/anaconda3] >>> 
```
最后会询问是否进行初始化，yes即可：
```
# 此处询问是否初始化conda的环境，直接输入yes
 
Do you wish the installer to initialize Anaconda3
by running conda init? 
 
[yes|no][no] >>> yes
```
初始化时将配置写入了./.bashrc文件中，输入下面命令就可以开始使用了：
```
source ./.bashrc
```
会看到命令行开头会出现(bash)，说明已经进入conda环境了。
要解除conda环境，只需```conda deactivate```即可。

## 2.如果机器上没有pip，安装pip3：
```
1.  yum -y install epel-release               
2.  yum -y install python3-pip                
3.  pip3 --version  查看pip版本
```

## 3.生成requirements.txt文件
这个文件用来记录所需要的环境和对应的版本号，最后使用pip读取该文件可以统一安装。
> 1. 到你的算法根目录下
> 2. 使用```pip freeze > requirements.txt```命令
> 
两步即可生成requirements.txt文件。


但是这样做的一个巨大问题是，它会将环境中所有的库名称和版本进行输出，有些库是在项目中没有用到的，但依然会进行输出。

为了避免这种情况，有人就开发了一个pipreqs库，它可以进行一些过滤，仅将工程中用到的库和版本进行输出。

pipreqs安装：
 ```
 pip install pipreqs
 ```
 安装好之后，在当前目录下运行：
```
pipreqs . --encoding=utf8 --force
```

稍等一会就生成好了。
我生成的requirements.txt内容如下：
```
alfred==0.3
config==0.5.1
coremltools==6.2
Flask==2.2.2
globals==0.3.35
helpers==0.2.0
Jinja2==3.0.3
matplotlib==3.5.2
nb==0.1.2
numpy==1.21.5
onnx==1.13.1
opencv_python==4.7.0.68
pandas==1.4.4
Pillow==9.4.0
PyYAML==6.0
requests==2.28.1
scikit_image==0.19.2
scipy==1.9.1
seaborn==0.11.2
signals==0.0.2
skimage==0.0
templating==0.5.0
thop==0.1.1.post2209072238
torch==1.9.1
torchvision==0.10.1
tqdm==4.64.1
Werkzeug==2.2.3
WTForms==3.0.1

```

## 4.安装依赖文件（执行此步之前建议先看后续踩坑部分，节省时间）

执行：
```
pip install -r ./requirements.txt
```
漫长的下载过程
## 5.踩的坑
### skimage报错
报错内容如下：
>  error: subprocess-exited-with-error
> 
>   × python setup.py egg_info did not run successfully.
>   │ exit code: 1
>   ╰─> [3 lines of output]
> 
>       *** Please install the `scikit-image` package (instead of `skimage`) ***
> 
>       [end of output]
> 
>   note: This error originates from a subprocess, and is likely not a problem with pip.
> error: metadata-generation-failed
> 
> × Encountered error while generating package metadata.
> ╰─> See above for output.
> 
> note: This is an issue with the package mentioned above, not pip.
> hint: See above for details.

看字面意思是说，不要安装skimage，而是安装scikit-image。
考虑到生成的requirements.txt文件中有这两行:
```
scikit_image==0.19.2
skimage==0.0
```
scikit-image已经安装好了，那直接把```skimage==0.0```删除即可。

### 安装成功后报dependency confilct错误
报错内容如下：
```
ERROR: pip's dependency resolver does not currently take into account all the packages that are installed. This behaviour is the source of the following dependency conflicts.
daal4py 2021.6.0 requires daal==2021.4.0, which is not installed.
anaconda-project 0.11.1 requires ruamel-yaml, which is not installed.
```

是有两个依赖```daal==2021.4.0```和```ruamel-yaml```没有安装，按照提示逐个安装：
```
pip install daal==2021.4.0
pip install ruamel-yaml
```

然而，在安装```daal==2021.4.0```这个库的时候，又给老子报了个错：
```
ERROR: Cannot uninstall 'TBB'. It is a distutils installed project and thus we cannot accurately determine which files belong to it which would lead to only a partial uninstall.
```
意思是安装过程中无法卸载老版本的TBB库。
那让我们友好地手动去卸载吧。
通过命令```pip install tbb```确定已安装的tbb库的位置。
我的输出如下：
```
Looking in indexes: https://mirrors.aliyun.com/pypi/simple
Requirement already satisfied: tbb in ./anaconda3/lib/python3.9/site-packages (0.2)
```
意思是包位置在./anaconda3/lib/python3.9/site-packages (0.2)，进入这个目录，执行```ls```，找到和tbb有关的结果：
```
tbb
TBB-0.2-py3.9.egg-info
TBB.py
```
把这三个删掉
```
rm -rf tbb TBB.py TBB-0.2-py3.9.egg-info
```
再去安装dall应该没问题了。
