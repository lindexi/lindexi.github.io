# Roslyn 在 msbuild 的 target 判断文件存在

在使用 msbuild 定义编译时运行的逻辑，可以使用 Exists 判断文件是否存在

<!--more-->
<!-- CreateTime:2019/12/2 8:31:05 -->

<!-- csdn -->

假设需要判断某个文件是否存在，如果存在则执行逻辑，如删除这个文件，可以使用下面代码

```csharp
        <PropertyGroup>
			<SourceProjectPackageFile>SourceProjectPackageFile.txt</SourceProjectPackageFile>
        </PropertyGroup>

		<Delete Files="$(SourceProjectPackageFile)" Condition="Exists($(SourceProjectPackageFile))"></Delete>
```

上面代码就可以用来删除定义的 SourceProjectPackageFile.txt 文件

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
