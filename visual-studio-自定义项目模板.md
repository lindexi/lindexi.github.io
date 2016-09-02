#Visual Studio 自定义项目模板

经常我们需要新建一个项目，然后新建我们的View文件夹，ViewModel文件夹，Model文件夹，还有把我们的ViewModelBase放入我们的VIewModel，如果还用框架，还需要加上好多。

而我一般还有用九幽统计，需要修改好多东西，每新建一个项目都要做这个，这样我觉得不好，在网上看到了自定义模板，不过垃圾微软官方说的好差，看不懂，看了老周的，还是觉得不懂，我就自己来。

我们需要打开我们目录：C:\Program Files (x86)\Microsoft Visual Studio 14.0\Common7\IDE\ProjectTemplates\CSharp\Windows Root\Windows UAP

里面有文件夹，一般我们打开最后一个，我也不知道你看到我这篇，垃圾微软把它改为最大多少，所以，一个一个来，我现在打开1033，（1033是老周博客写的）我的其实还有更后的，这个如果都是数字，就选最大的。

我们先把文件夹复制到我们用户文档或者自己程序的项目位置，然后压缩一份保存，因为怕自己弄坏

然后我们用Visual Studio打开文件，记住，要打开.csproj要用文件 打开。

我们先打开BlankApplication里BlankApplication.vstemplate

我们要修改