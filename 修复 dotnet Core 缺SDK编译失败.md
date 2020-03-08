# 修复 dotnet Core 缺SDK编译失败

在打开一个 sln 项目包含 dotnet core 的时候，可能在打开的时候提示找不到 sdk 一般是没有在安装的时候安装对应的开发

<!--more-->
<!-- CreateTime:2019/6/23 10:55:09 -->


如果在导入一个 sln 文件的时候看到下面的提示

```
.NETCore SDK Not Found
```

![](http://image.acmx.xyz/5c82777e-6e2b-4d9b-a07f-5d83e2ae2cd7201612485939.jpg)

需要判断现在使用是Vs2015或VS2017版本

如果是2015，去 [https://github.com/aspnet/Tooling/blob/master/known-issues-vs2015.md](https://github.com/aspnet/Tooling/blob/master/known-issues-vs2015.md) 下载对应版本

如果是2017，去 [https://github.com/aspnet/Tooling/blob/master/known-issues-vs2017.md](https://github.com/aspnet/Tooling/blob/master/known-issues-vs2017.md)  下载

当然不想下载，修改`global.json`文件，将使用的版本号修改为当前你的设备已经安装的版本

```
{
  "projects": [ "src", "test" ],
  "sdk": 
  {
    "version": "1.0.0-preview2-003121"  你需要的版本
  }
}
```

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://i.creativecommons.org/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
