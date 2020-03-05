# 在 windows 安装 Jekyll

本文告诉大家一个简单的方法在 Windows 安装 Jekyll

<!--more-->
<!-- CreateTime:2018/2/22 17:47:39 -->

<div id="toc"></div>

## 下载 ps1 文件

首先需要安装 Chocolatey ，这个工具可以快速安装 Jekyll

先下载[Chocolatey](https://chocolatey.org/install.ps1)，如果无法从这个地方下载，请到csdn[下载](http://download.csdn.net/download/lindexi_gd/10132718 )

然后管理员打开 PowerShell ，因为需要使用脚本，所以让 PowerShell 支持脚本

```csharp
Set-ExecutionPolicy bypass
```

需要注意 bypass 是不需要签名就可以运行脚本，对于开发者，这个功能是很好的，但是如果你不是开发者，那么请不要使用这个功能。

```csharp
如果不是开发者，请用下面代码

 Set-ExecutionPolicy bypass process
```

## 安装 Chocolatey 

把下载的脚本放到 PowerShell 运行就可以

然后输入下面代码

```csharp
 SET "PATH=%PATH%;%ALLUSERSPROFILE%\chocolatey\bin"
```

## 安装 Ruby

使用下面的命令安装

```csharp
choco install ruby -y
```

## 安装bundler

使用下面的命令

```csharp
cd  C:\tools\ruby24\bin
 .\gem install bundler
```

## 安装Jekyll

使用下面命令安装

```csharp
.\gem install jekyll

.\gem install jekyll bundler
```

然后重新打开命令行输入

```csharp
bundle install
```

这样就可以安装了，使用下面的代码启动

```csharp
jekyll new myblog
cd myblog
jekyll serve
```

感谢

[Easily install Jekyll on Windows with 3 command prompt entries and Chocolatey ](https://davidburela.wordpress.com/2015/11/28/easily-install-jekyll-on-windows-with-3-command-prompt-entries-and-chocolatey/ )

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。  