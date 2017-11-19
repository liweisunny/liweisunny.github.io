---
title: setuptools和pip安装教程
date: 2017-11-16 16:03:08
tags: python环境部署
---
# 安装setuptools
①下载setuptools源码**setuptools-25.2.0.tar.gz**，地址：[https://pypi.python.org/pypi/setuptools](https://pypi.python.org/pypi/setuptools "setuptools下载") 
![](https://i.imgur.com/a29kEga.png)
②这是一个压缩文件，将其解压到桌面，并进入该文件夹；按住shift键后，在文件夹空白处点击鼠标右键，选择：在此处打开命令窗口，输入：
	  
```bash
  python setup.py install
```
回车，看到如下图内容即表示安装成功：
![](https://i.imgur.com/epT3f5n.png)

③安装完后在当前窗口中输入 **easy_install** 回车，进行检测，如果提示： 
**error: No urls, filenames, or requirements specified (see –help**) 说明安装成功，它在提示你命令后面需要跟参数。 如果提示： **‘easy_install’** 不是内部或外部命令，也不是可运行的程序或批处理文件。 请检查系统环境变量path是否配置了**‘C:\Python27\Scripts’**
# 安装pip
## 使用easy_install安装pip
如果setuptools安装好后，可以直接用easy_install来安装pip，如下图：
![](https://i.imgur.com/I0axT0o.png)
## 手动安装pip
如果还想手动安装的话，和安装setuptools步骤完全一样

①下载pip压缩包**pip-9.0.1.tar.gz**，地址：[https://pypi.python.org/pypi/pip](https://pypi.python.org/pypi/pip "pip下载")，如下图：![](https://i.imgur.com/gGKpfRK.png)

②这是一个压缩文件，将其解压到桌面，并进入该文件夹，按住shift键后，在文件夹空白处点击鼠标右键，选择：在此处打开命令窗口，输入：
	  
```bash
python setup.py install 
```

回车，看到如下图内容即表示安装成功：
![](https://i.imgur.com/vZev7uY.png)

③安装成功后输入 **pip** 回车，进行检测如果提示： **Did not provide a command** 说明安装成功，因为pip后面也需要跟参数；如果提示： **‘pip’ **不是内部或外部命令，也不是可运行的程序或批处理文件，请检查环境变量path是否配置了**‘C:\Python27\Scripts’**

# 补充
卸载pip:

```bash
python -m pip uninstall pip setuptools
```

升级pip:

```bash
python -m pip install --upgrade pip
```

