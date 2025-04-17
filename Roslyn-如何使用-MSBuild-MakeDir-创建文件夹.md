
# Roslyn 如何使用 MSBuild MakeDir 创建文件夹

本文告诉大家如何在 MSBuild 里使用 MakeDir 创建文件夹

<!--more-->


<!-- CreateTime:2020/11/15 19:26:49 -->

<!-- 标签：Roslyn,MSBuild,编译器 -->

在 MSBuild 的 Task 内置任务里面，可以使用 MakeDir task 进行创建文件夹，简单的使用方法如下

```xml
<Project xmlns="http://schemas.microsoft.com/developer/msbuild/2003">

    <PropertyGroup>
        <OutputDirectory>\Output\</OutputDirectory>
    </PropertyGroup>

    <Target Name="CreateDirectories">
        <MakeDir Directories="$(OutputDirectory)"/>
    </Target>

</Project>

```

上面代码中，核心逻辑就是 `<MakeDir Directories="$(OutputDirectory)"/>` 用来创建文件夹。有多个文件夹，可以通过 `;` 分割

在 MakeDir task 里面还有一个属性是 `DirectoriesCreated` 属性，这个属性用来输出表示有哪些文件夹创建成功的。也就是说在 Directories 属性里面传入的文件夹列表里面，所有创建成功的都会在 `DirectoriesCreated` 属性输出

读取输出的创建成功的文件夹代码如下

```xml
<Target Name="_WalterlvCreateDirectoryForPacking">
    <MakeDir Directories="$(MSBuildThisFileDirectory)..\bin\$(Configuration)\">
        <Output TaskParameter="DirectoriesCreated" PropertyName="CreatedPackingDirectory" />
    </MakeDir>
</Target>
```

判断文件夹不存在，则创建文件夹的代码可以是如下

```xml
    <PropertyGroup>
        <OutputDirectory>\Output\</OutputDirectory>
    </PropertyGroup>

    <Target Name="CreateDirectories">
        <MakeDir Condition="!Exists('$(OutputDirectory)')"
                 Directories="$(OutputDirectory)" />
    </Target>
```

更多在 MSBuild 编译过程中操作文件和文件夹的细节请看 [在 MSBuild 编译过程中操作文件和文件夹（检查存在/创建文件夹/读写文件/移动文件/复制文件/删除文件夹）walterlv - 吕毅-CSDN博客](https://walterlv.blog.csdn.net/article/details/103760615)

更多请看官方文档 [MakeDir Task - Visual Studio](https://docs.microsoft.com/en-us/visualstudio/msbuild/makedir-task?view=vs-2019)

更多关于 Roslyn 请看 [手把手教你写 Roslyn 修改编译](https://blog.lindexi.com/post/roslyn.html ) 

参见：[Roslyn 入门 - CSDN博客](https://blog.csdn.net/lindexi_gd/category_7945110.html )





<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。