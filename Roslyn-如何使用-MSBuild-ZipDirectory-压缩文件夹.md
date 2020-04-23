
# Roslyn 如何使用 MSBuild ZipDirectory 压缩文件夹

在 csproj 文件或在 NuGet 的 Targets 文件中可以通过 Target 调用 ZipDirectory 任务用来制作压缩包，在构建的时候，可以用这个方法将某个输出文件夹等内容压缩输出

<!--more-->


<!-- CreateTime:4/21/2020 7:50:26 PM -->

<!-- 发布 -->

使用 ZipDirectory 有两个必要的属性，一个是 DestinationFile 表示输出的 zip 文件的路径，另一个是 SourceDirectory 表示将被压缩的文件夹路径

如果 DestinationFile 文件期望进行覆盖，也就是如果 DestinationFile 路径已经存在，将覆盖写入新的 zip 文件，可以使用 Overwrite 属性

使用方法如下

```xml
    <Target Name="ZipOutputPath" AfterTargets="Build">
        <ZipDirectory
            SourceDirectory="$(OutputPath)"
            DestinationFile="$(MSBuildProjectDirectory)\lindexi.zip" />
    </Target>
```

将上面代码放在 csproj 文件，构建将会在 csproj 文件所在文件夹找到创建的文件

本文代码放在[github](https://github.com/lindexi/lindexi_gd/tree/c55f0a334b5eac0cdd3c12046961af8573f76369/BerjearnearheliCallrachurjallhelur)欢迎小伙伴访问

[ZipDirectory Task](https://docs.microsoft.com/en-us/visualstudio/msbuild/zipdirectory-task?view=vs-2019 )





<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。