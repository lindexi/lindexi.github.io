# BAT 脚本判断当前系统是 x86 还是 x64 系统

本文告诉大家在写 BAT 脚本的时候，如何判断当前的系统是 32 位系统的还是 64 位系统

<!--more-->
<!-- CreateTime:2019/8/30 8:47:40 -->


通过注册表进行判断方法

```csharp
@echo OFF

reg Query "HKLM\Hardware\Description\System\CentralProcessor\0" | find /i "x86" > NUL && set OS=32BIT || set OS=64BIT

if %OS%==32BIT 在这里执行 32 位系统代码
if %OS%==64BIT 在这里执行 64 位系统代码
```

如判断系统然后输出

```csharp
@echo OFF

reg Query "HKLM\Hardware\Description\System\CentralProcessor\0" | find /i "x86" > NUL && set OS=32BIT || set OS=64BIT

if %OS%==32BIT echo This is a 32bit operating system
if %OS%==64BIT echo This is a 64bit operating system
```

第二个方法是通过 `Program Files (x86)` 文件夹存在判断方法

```csharp
:CheckOS
IF EXIST "%PROGRAMFILES(X86)%" (GOTO 64BIT) ELSE (GOTO 32BIT)

:64BIT
echo 64-bit...
在这里执行 64 位系统代码
GOTO END

:32BIT
echo 32-bit...
在这里执行 32 位系统代码
GOTO END

:END
```

[windows - batch file to check 64bit or 32bit OS - Stack Overflow](https://stackoverflow.com/questions/12322308/batch-file-to-check-64bit-or-32bit-os )

[How To Check If Computer Is Running A 32 Bit or 64 Bit Operating System](https://support.microsoft.com/en-us/help/556009 )

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。        
