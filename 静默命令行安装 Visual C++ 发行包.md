# 静默命令行安装 Visual C++ 发行包

本文告诉大家如何通过命令行的方法，在安装程序静默调用 VC++ 库的安装，解决缺少环境问题

<!--more-->
<!-- CreateTime:2019/8/27 15:39:03 -->

<!-- csdn -->

对不同的版本的 VC++ 库安装的方法有所不同，每个版本的 VC++ 库都会有 x86 x64 ia64 等版本，本文将使用 x86 版本作为例子，而对应的其他版本安装方法相同

## 安装

### Visual C++ 2017

x86

```csharp
vc_redist.x86.exe /install /quiet /log "%temp%\Install_vc_redist_2017_x86.log"
```

x64

```csharp
vc_redist.x64.exe /install /quiet /log "%temp%\Install_vc_redist_2017_x64.log"
```

### Visual C++ 2015

```csharp
vc_redist.x86.exe /q /norestart
```

### Visual C++ 2013

```csharp
vcredist_x86.exe /install /quiet /norestart /log %TEMP%\vcredist_2013_x86.log
```

### Visual C++ 2010

```csharp
// x86
vcredist_x86.exe /q /norestart
// x64
vcredist_x64.exe /q /norestart
// ia64
vcredist_ia64.exe /q /norestart
```

### Visual C++ 2008

```csharp
vcredist_x86.exe /q
```

### Visual C++ 2005

这里的 Visual C++ 2005 也就是 Visual C++ 8.0 版本，静默安装方法请看下面

```csharp
Vcredist_x86.exe /q:a /c:"msiexec /i vcredist.msi /qn /l*v %temp%\vcredist_x86.log"
```

## 卸载

### Visual C++ 2017

```csharp
vc_redist.x86.exe /uninstall /quiet /log "%temp%\Uninstall_vc_redist_2017_x86.log"
```

如果找不到 `vc_redist.x86.exe` 可以尝试从缓存文件找到

```csharp
x86 : "C:\ProgramData\Package Cache\{2019b6a0-8533-4a04-ac0e-b2c10bdb9841}\VC_redist.x86.exe" /uninstall /quiet
x64 : "C:\ProgramData\Package Cache\{80586c77-db42-44bb-bfc8-7aebbb220c00}\VC_redist.x64.exe" /uninstall /quiet
```

### Visual C++ 2010 

```csharp
vcredist_x86.exe /q /uninstall /norestart
```

### Visual C++ 2008

普通的卸载

```csharp
vcredist_x86.exe /qb
```

不带取消的卸载

```csharp
vcredist_x86.exe /qb!
``` 

### Visual C++ 2005

```csharp
vcredist_x86.exe /q:a /c:"msiexec /i vcredist.msi /qb! /l*v %temp%\vcredist_x86.log"
```

## 下载地址

- [Visual C++ 2017 Redistributable Package (x86)](https://aka.ms/vs/15/release/vc_redist.x86.exe)
- [Visual C++ 2017 Redistributable Package (x64)](https://aka.ms/vs/15/release/vc_redist.x64.exe)

- [Microsoft Visual C++ 2015 Redistributable (x86)](https://download.microsoft.com/download/9/3/F/93FCF1E7-E6A4-478B-96E7-D4B285925B00/vc_redist.x86.exe)
- [Download Visual C++ Redistributable Packages for Visual Studio 2013 from Official Microsoft Download Center](https://www.microsoft.com/en-us/download/details.aspx?id=40784 )
- [Visual C++ 2010 Redistributable Package (x86)](http://www.microsoft.com/downloads/details.aspx?FamilyID=a7b7a05e-6de6-4d3a-a423-37bf0912db84)
- [Visual C++ 2010 Redistributable Package (x64)](http://www.microsoft.com/downloads/details.aspx?familyid=BD512D9E-43C8-4655-81BF-9350143D5867)
- [Visual C++ 2010 Redistributable Package (ia64)](http://www.microsoft.com/downloads/details.aspx?FamilyID=1a2df53a-d8f4-4bfe-be35-152c5d3d0f82)

- [VC 8.0 Visual C++ 2005 Redistributable Package (x86)](http://www.microsoft.com/downloads/details.aspx?familyid=32BC1BEE-A3F9-4C13-9C99-220B62A191EE&displaylang=en)
- [VC 8.0 Visual C++ 2005 Redistributable Package (x64)](http://www.microsoft.com/downloads/details.aspx?familyid=90548130-4468-4bbc-9673-d6acabd5d13b&displaylang=en)
- [VC 8.0 Visual C++ 2005 Redistributable Package (ia64)](http://www.microsoft.com/downloads/details.aspx?FamilyID=747AAD7C-5D6B-4432-8186-85DF93DD51A9&displaylang=en)

[IT Pro Tips for Microsoft Visual C++ Redistribuable 2017 v14](https://www.itninja.com/software/microsoft/visual-c-redistribuable-2017/v14-1 )

[Mailbag: How to perform a silent install of the Visual C++ 2010 redistributable packages – Aaron Stebner's WebLog](https://blogs.msdn.microsoft.com/astebner/2010/10/20/mailbag-how-to-perform-a-silent-install-of-the-visual-c-2010-redistributable-packages/ )

[Mailbag: How to perform a silent install of the Visual C++ 2008 redistributable packages – Aaron Stebner's WebLog](https://blogs.msdn.microsoft.com/astebner/2009/03/27/mailbag-how-to-perform-a-silent-install-of-the-visual-c-2008-redistributable-packages/ )

[Update regarding silent install of the VC 8.0 runtime (vcredist) packages – Aaron Stebner's WebLog](https://blogs.msdn.microsoft.com/astebner/2007/02/07/update-regarding-silent-install-of-the-vc-8-0-runtime-vcredist-packages/ )

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。        
