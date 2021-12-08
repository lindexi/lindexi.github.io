# dotnet 6 引用 NAudio 的旧版本构建不通过

本文告诉大家在使用 NAudio 的旧版本导致构建不通过问题，解决方法是升级到 1.10 或以上版本

<!--more-->
<!-- CreateTime:2021/12/6 19:43:07 -->

<!-- 发布 -->

在更新 dotnet 6 项目时，使用了 NAudio 的旧版本，构建失败，提示 MC1000 如下

```
未知的生成错误“Could not find assembly 'System.Private.CoreLib, Version=4.0.0.0, Culture=neutral, PublicKeyToken=7cec85d7bea7798e'. Either explicitly load this assembly using a method such as LoadFromAssemblyPath() or use a MetadataAssemblyResolver that returns a valid assembly.”
```

提示的文件如下

```
C:\Program Files\dotnet\sdk\6.0.100\Sdks\Microsoft.NET.Sdk.WindowsDesktop\targets\Microsoft.WinFX.targets
```

解决方法是升级到 1.10 或以上版本

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
