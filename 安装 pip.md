# 安装 pip

本文告诉大家如何在 VisualStudio 下安装 Python 安装 Pip

<!--more-->
<!-- CreateTime:2018/3/5 19:04:04 -->

<!-- csdn -->

<!-- 标签：软件，pip，python -->

首先用VisualStudio安装Python然后使用下面的代码可以获得安装的 Python 的路径

```csharp
>>> import sys
>>> path = sys.executable
>>> print(path)
```

打开 powershell 然后输入 py ，然后输入上面的代码

```csharp
Python 3.6.3 (v3.6.3:2c5fed8, Oct  3 2017, 18:11:49) [MSC v.1900 64 bit (AMD64)] on win32
Type "help", "copyright", "credits" or "license" for more information.
>>> import sys
>>> path = sys.executable
>>> print(path)
C:\Program Files (x86)\Microsoft Visual Studio\Shared\Python36_64\python.exe
```

可以看到python在哪，然后下载[get-pip.py](https://bootstrap.pypa.io/get-pip.py)

使用 Powershell 打开 get-pip.py 所在的路径然后输入 `py get-pip.py`就可以

安装完成打开 python 安装下的 Scripts 就可以看到 pip 这时复制路径，打开环境变量，设置path就可以使用

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。