
# dotnet Roslyn 通过读取 suo 文件获取解决方案的启动项目

本文来告诉大家一个黑科技，通过 .suo 文件读取 VisualStudio 的启动项目。在 sln 项目里面，都会生成对应的 suo 文件，这个文件是 OLE 格式的文件，文件的格式没有公开，本文的方法适合用在 VisualStudio 2019 上，对于其他版本的 VisualStudio 也许会不适合

<!--more-->


<!-- CreateTime:2021/4/28 20:39:45 -->


<!-- 发布 -->
<!-- 标签：Roslyn,MSBuild,编译器 -->

感谢 [Simon Cropp](https://github.com/SimonCropp) 大佬提供的方法

默认在 sln 解决方案文件的相同文件夹里面，将会存放 `.vs\{解决方案名}\v{VS版本}\.suo` 文件，如解决方案文件名为 `HairhechallchujurKairbilairlem.sln` 在 VisualStudio 2019 下将会存放 `.vs\HairhechallchujurKairbilairlem\v16\.suo` 文件

这个 `.suo` 文件是包含了 VisualStudio 解决方案的一些配置，如启动项目。关多关于此文件，请参阅 [Solution User Options (.Suo) File 文档](https://docs.microsoft.com/zh-cn/previous-versions/visualstudio/visual-studio-2015/extensibility/internals/solution-user-options-dot-suo-file?view=vs-2015&WT.mc_id=WD-MVP-5003260 )

预计这个 suo 格式文件基本不会更改，在 1995 年的时候就开始使用这个格式

读取 .suo 需要使用到 [Open MCDF](https://github.com/ironfede/openmcdf) 库。这是一个完全由 C# 实现的读取 OLE 格式文档的库，我在做 OFFICE 组件也用到这个库

在 suo 文件里面，通过 SolutionConfiguration 内容存放当前的启动项，这里面的内容是使用 UTF-16 编码的字符串，读取的方法如下

```csharp
            using (var fileStream = new FileStream(suoFilePath, FileMode.Open))
            {
                using CompoundFile compoundFile = new CompoundFile(fileStream, CFSUpdateMode.ReadOnly, CFSConfiguration.SectorRecycle | CFSConfiguration.EraseFreeSectors);
                var cfStream = compoundFile.RootStorage.GetStream("SolutionConfiguration");
                var byteList = cfStream.GetData();
                var encoding = Encoding.GetEncodings()
                    .Single(x => string.Equals(x.Name, "utf-16", StringComparison.OrdinalIgnoreCase));
                var text = encoding.GetEncoding().GetString(byteList);
            }
```

这里的 text 的内容大概如下

```
"\u0011\0MultiStartupProj\0=\u0003\0\0;4\0{45171CDC-EDAC-4D0B-BDF8-63DE2D4F947B}.dwStartupOpt\0=\u0003\0\0;\u000f\0StartupProject\0=\b&\0{45171CDC-EDAC-4D0B-BDF8-63DE2D4F947B};A\0{45171CDC-EDAC-4D0B-BDF8-63DE2D4F947B}.Release|Any CPU.fBatchBld\0=\u0003\0\0;?\0{45171CDC-EDAC-4D0B-BDF8-63DE2D4F947B}.Debug|Any CPU.fBatchBld\0=\u0003\0\0;4\0{AE3577E5-5D4E-44F8-B181-88A31B92584A}.dwStartupOpt\0=\u0003\0\0;A\0{AE3577E5-5D4E-44F8-B181-88A31B92584A}.Release|Any CPU.fBatchBld\0=\u0003\0\0;?\0{AE3577E5-5D4E-44F8-B181-88A31B92584A}.Debug|Any CPU.fBatchBld\0=\u0003\0\0;4\0{A2FE74E1-B743-11D0-AE1A-00A0C90FFFC3}.dwStartupOpt\0=\u0003\0\0;\n\0ActiveCfg\0=\b\r\0Debug|Any CPU;"
```

通过读取 StartupProject 后续的内容即可找到当前的启动项目的 GUID 值，以下是我写的正则

```csharp
                var text = encoding.GetEncoding().GetString(byteList);

                const char nul = '\u0000';
                const char dc1 = '\u0011';
                const char etx = '\u0003';
                const char soh = '\u0001';

                var startupProjectRegex = new Regex(@$"StartupProject{nul}={'\b'}&{nul}(.{'{'}{38}{'}'});A");
                var startupProjectMatch = startupProjectRegex.Match(text);
                if (startupProjectMatch.Success)
                {
                    var guid = Guid.Parse(startupProjectMatch.Groups[1].Value);
                }
```

上面代码拿到的 guid 就是启动项目的 guid 内容

咱可以采用 [Simon Cropp](https://github.com/SimonCropp) 大佬的开源项目 https://github.com/SimonCropp/SetStartupProjects 来辅助读取当前 sln 里面包含的 csproj 的 GUID 和路径

代码如下

```csharp
var projectList = SetStartupProjects.SolutionProjectExtractor.GetAllProjectFiles(solutionFile.FullName).ToList();
```

通过 guid 获取当前的 csproj 项目文件路径方法如下

```csharp
                    var guid = Guid.Parse(startupProjectMatch.Groups[1].Value);
                    var project = projectList.FirstOrDefault(temp => new Guid(temp.Guid) == guid);
```

我封装了方法，传入的是 sln 文件，返回启动项目的路径

```csharp
        private static FileInfo GetStartupProject(FileInfo solutionFile)
        {
            var solutionFilePath = solutionFile.FullName;
            var solutionDirectory = solutionFile.DirectoryName;

            var solutionName = Path.GetFileNameWithoutExtension(solutionFilePath);
            var suoDirectoryPath = Path.Combine(solutionDirectory, ".vs", solutionName, "v16");
            Directory.CreateDirectory(suoDirectoryPath);
            var suoFilePath = Path.Combine(suoDirectoryPath, ".suo");

            var projectList = SetStartupProjects.SolutionProjectExtractor.GetAllProjectFiles(solutionFile.FullName).ToList();
            using (var fileStream = new FileStream(suoFilePath, FileMode.Open))
            {
                using CompoundFile compoundFile = new CompoundFile(fileStream, CFSUpdateMode.ReadOnly, CFSConfiguration.SectorRecycle | CFSConfiguration.EraseFreeSectors);
                var cfStream = compoundFile.RootStorage.GetStream("SolutionConfiguration");
                var byteList = cfStream.GetData();
                var encoding = Encoding.GetEncodings()
                    .Single(x => string.Equals(x.Name, "utf-16", StringComparison.OrdinalIgnoreCase));
                var text = encoding.GetEncoding().GetString(byteList);

                const char nul = '\u0000';
                const char dc1 = '\u0011';
                const char etx = '\u0003';
                const char soh = '\u0001';

                var startupProjectRegex = new Regex(@$"StartupProject{nul}={'\b'}&{nul}(.{'{'}{38}{'}'});A");
                var startupProjectMatch = startupProjectRegex.Match(text);
                if (startupProjectMatch.Success)
                {
                    var guid = Guid.Parse(startupProjectMatch.Groups[1].Value);
                    var project = projectList.FirstOrDefault(temp => new Guid(temp.Guid) == guid);
                    return new FileInfo(project.FullPath);
                }
            }

            return null;
        }
```

需要先在项目安装 [SetStartupProjects](https://www.nuget.org/packages/SetStartupProjects/ ) 库，才能使用这个方法

本文所有代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/8560b967e202124f78e2cb47c0279b48bffb1cb5/HairhechallchujurKairbilairlem) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/8560b967e202124f78e2cb47c0279b48bffb1cb5/HairhechallchujurKairbilairlem) 欢迎小伙伴访问


除了读取启动项目，还可以读取断点等内容，读取 suo 里面的所有内容的方法如下

```csharp
                compoundFile.RootStorage.VisitEntries(item =>
                {
                    if (item.IsStream)
                    {
                        Console.WriteLine(item.Name);

                        var stream = item as CFStream;
                        byteList = stream.GetData();
                        text = encoding.GetEncoding().GetString(byteList);
                    }
                }, true);
```

当然了，获取到的内容不一定使用 UTF-16 编码格式，还需要自己尝试，里面的数据只是二进制而已，上面代码的转换字符串只是用来调试

更多请看

[SimonCropp/SetStartupProjects: Setting Visual Studio startup projects by hacking the suo](https://github.com/SimonCropp/SetStartupProjects )

[Solution User Options (.Suo) File](https://docs.microsoft.com/zh-cn/previous-versions/visualstudio/visual-studio-2015/extensibility/internals/solution-user-options-dot-suo-file?view=vs-2015&WT.mc_id=WD-MVP-5003260 )

更多编译相关请看[手把手教你写 Roslyn 修改编译](https://blog.lindexi.com/post/roslyn.html )





<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。