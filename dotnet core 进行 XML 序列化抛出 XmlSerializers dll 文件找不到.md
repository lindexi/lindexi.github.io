# dotnet core 进行 XML 序列化抛出 XmlSerializers dll 文件找不到

在将原本的 dotnet framework 版本的 WPF 项目迁到 dotnet core 版本，在进行单元测试，发现在 XmlSerializer 抛出了 System.Private.CoreLib.XmlSerializers.dll 找不到的异常，其实这个只是在 XmlSerializer 的内部异常，可以忽略

<!--more-->
<!-- CreateTime:2020/9/24 20:27:49 -->

<!-- 发布 -->

在 dotnet core 下，使用如下代码进行 xml 序列化，其中 Foo 是我定义的类

```csharp
   var xmlSerializer = new XmlSerializer(typeof(Foo));
```

应用将会在 VS 打开所有异常的时候，可以看到如下代码

```
System.IO.FileNotFoundException:“Could not load file or assembly 'C:\Users\lindexi\.nuget\packages\microsoft.testplatform.testhost\16.5.0\build\netcoreapp2.1\x64\System.Private.CoreLib.XmlSerializers.dll'. 系统找不到指定的文件。”
```

也就是 `System.Private.CoreLib.XmlSerializers.dll` 找不到的异常，其实在 .NET Core 的这个异常只是在 XmlSerializer 内部抛出，会被 XML 框架接住，上层啥都不需要做

因此，只需要忽略就可以

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
