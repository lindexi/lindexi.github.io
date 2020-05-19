# MSBuild 常用参数

本文告诉大家在 MSBuild 里面常用的参数

<!--more-->
<!-- CreateTime:2019/12/4 18:56:42 -->


一般的 msbuild 在编译的时候都会添加很多参数，用法如下

进入对应编译的 sln 或 csproj 文件所在的文件夹，执行下面命名

```csharp
msbuild 
```

如果在文件夹里面存在多个不同的 sln 文件等，在 msbuild 后面添加对应的文件

```csharp
msbuild xx.sln
```

此外添加的参数写在 msbuild 后面

## 并行编译

有多个项目一起编译，通过并行编译可以加快速度

用 `/m` 参数开启并行编译

```csharp
msbuild /m
```

通过后面带上数字表示多少 CPU 同时编译，下面代码表示 6 个 CPU 同时编译

```csharp
msbuild /m:6
```

## 发布版编译

通过发布版编译就是设置 configuration 属性为 release 通过下面代码

```csharp
msbuild /p:Configuration=Release
```

在msbuild通过 `/p` 设置对应的属性的值

在 msbuild 的参数是不区分 `/` 和 `-` 也就是 `/p` 和 `-p` 是相同

## 重新编译

通过 `-t:rebuild` 重新编译

```csharp
msbuild -t:rebuild 
```

## 日志

通过 `-fileLogger` 或 `-fl` 可以指定输出到文件，通过 `-filelogparameters` 或短参数 `flp` 可以指定输出的日志文件

```csharp
msbuild -fl -flp:logfile=xx.log;verbosity=n
``` 

这里的 verbosity 表示输出等级

运行上面代码建议编译当前文件夹里面的项目，然后将编译日志输出到 xx.log 文件夹

## 执行包还原

通过 `-t:restore` 可以还原包

```csharp
msbuild -t:restore
```

建议的还原方法是下面代码

```csharp
 NuGet restore
 dotnet restore
 msbuild -t:restore
```

## 清理项目

通过 `-t:clean` 清理项目

```csharp
msbuild -t:clean
```

## 打包

通过 `/t:pack` 打包

```csharp
msbuild -t:pack
```

## UWP 打包

```csharp
msbuild /t:restore /t:Publish /p:Configuration=Release /p:AppxPackageDir="D:\lindexi\AppxPackages\\" /p:AppxBundle=Always /p:UapAppxPackageBuildMode=StoreUpload /p:AppxBundlePlatforms="x86|x64|arm"
```

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
