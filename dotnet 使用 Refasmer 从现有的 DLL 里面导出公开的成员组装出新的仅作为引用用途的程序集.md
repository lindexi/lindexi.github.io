# dotnet 使用 Refasmer 从现有的 DLL 里面导出公开的成员组装出新的仅作为引用用途的程序集

本文告诉大家 Resharper 家（JetBrains）开源的 Refasmer 工具，这个工具的功能就是从现有的 DLL 或 EXE 程序集里面，将所有的公开成员导出，重新打出新的 DLL 或 EXE 程序集。此时打出的新的程序集里面的方法都不包含实现，而是只有定义，这样的程序集被成为仅作为引用用途的程序集，也就是 Reference assemblies 的意思，这也就是 Refasmer 命名来源

<!--more-->
<!-- CreateTime:2021/7/9 8:38:25 -->

<!-- 发布 -->

这个 Refasmer 工具是是一个 dotnet tool 工具，安装非常方便，使用如下代码进行安装

```
dotnet tool install -g JetBrains.Refasmer.CliTool
```

接着咱用一个现有的 DLL 来测试一下用途，如我有一个叫 Lindexi.dll 文件，我采用如下命令行进行转换为仅作为引用用途的程序集

```
refasmer Lindexi.dll
```

输入上面代码之后，将会在相同的文件夹下生成 `Lindexi.dll.refasm.dll` 文件，这个文件就是引用文件啦，不需要通过 dnspy 只通过文件的大小即可看出两个 DLL 的不同。在 dnspy 上可以看到 refasm.dll 里面只有方法的定义和属性的定义，但是没有任何具体的实现

这样的仅作为引用用途的程序集可以方便用在某些插件开发 SDK 或者某些不支持的版本上，用于构建用途。或者用在某些超大项目上，用于支持超大解决方案里面单个项目的独立构建

和 dotnet 自带的 ProduceOnlyReferenceAssembly 不同之处在于 Refasmer 可以从 DLL 里面导出仅作为引用用途的程序集。但 ProduceOnlyReferenceAssembly 是需要从源代码里面生成，更多关于 ProduceOnlyReferenceAssembly 请看 [msbuild 使用 ProduceOnlyReferenceAssembly 创建作为引用的仅公开成员程序集](https://blog.lindexi.com/post/msbuild-%E4%BD%BF%E7%94%A8-ProduceOnlyReferenceAssembly-%E5%88%9B%E5%BB%BA%E4%BD%9C%E4%B8%BA%E5%BC%95%E7%94%A8%E7%9A%84%E4%BB%85%E5%85%AC%E5%BC%80%E6%88%90%E5%91%98%E7%A8%8B%E5%BA%8F%E9%9B%86.html )

除了以上默认命令外，还可以给 refasmer 加上以下参数，用于实现更多功能

## 指定导出文件夹

默认的命令导出的是 `refasm.dll` 文件，显然这样的文件是不适合作为 NuGet 重新发布的，否则后续将会因为找不到对应的 DLL 而让运行时失败。但是如果文件命名上和之前的 DLL 相同，又会覆盖现有的文件

解决方法就是给定输出文件夹，通过 `-O` 参数，如下面代码

```
refasmer Lindexi.dll -O foo
```

运行完成上面代码，可以在 foo 文件夹里面，找到 `Lindexi.dll` 文件，在这个 foo 文件夹里面的 dll 文件是仅作为引用用途的程序集

另外，如果只是想输出单个文件的，可以通过 `-o` 参数，这个参数用于重新设置文件名，如下面命令

```
refasmer Lindexi.dll -o LindexiDoubi.dll
```

执行以上代码，将会输出 `LindexiDoubi.dll` 程序集

## 安静运行

默认的 refasmer 也没啥输出，如果不想要有任何输出，请加入 `-q` 参数，如下面命令

```
refasmer Lindexi.dll -O foo -q
```

## 输出调试信息

和 安静运行 不同的是，加上 `-v` 参数，输出更多信息

```
refasmer Lindexi.dll -O foo -v
```

## 列举文件信息

加上 `-l` 参数或 `--list` 参数可以输出本次导出的文件信息，如下面命令

```
refasmer Lindexi.dll -l
```

输出内容大概如下

```csharp
<?xml version="1.0" encoding="Codepage - 936"?>
<FileList>
  <File AssemblyName="Lindexi" Version="1.1.0.0" Culture="neutral" PublicKeyToken="0902d2af90156091" InGac="false" ProcessorArchitecture="MSIL" />
</FileList>
```

更多命令和使用方法请看 [JetBrains/Refasmer: The tool to create reference assembly from common assembly.](https://github.com/JetBrains/Refasmer )

## 自定义

如果发现当前的命令行不符合需求，还可以自己动手写代码

先创建一个 .NET 控制台项目，接着通过 NuGet 安装 JetBrains.Refasmer 库

```xml
  <ItemGroup>
    <PackageReference Include="JetBrains.Refasmer" Version="1.0.12" />
  </ItemGroup>
```

下面以创建此控制台项目的仅作为引用用途的程序集做例子，代码如下

```csharp
        static void Main(string[] args)
        {
            var file = Assembly.GetExecutingAssembly().Location;
            var output = file + Path.GetRandomFileName();

            ToReferenceAssembly(file, output);
        }

        private static void ToReferenceAssembly(string file, string output)
        {
            using var peReader = new PEReader(new FileStream(file, FileMode.Open, FileAccess.Read, FileShare.Read));
            var metaReader = peReader.GetMetadataReader();
            if (metaReader.IsAssembly)
            {
                var result =
                    MetadataImporter.MakeRefasm(metaReader, peReader, new LoggerBase(new VerySimpleLogger(Console.Out)));

                File.WriteAllBytes(output, result);
            }
        }
```

以上代码通过传入 ToReferenceAssembly 方法给定 DLL 程序集和期望输出的文件路径，然后将会执行 Refasmer 库提供的方法，从 file 里面读取公开成员，写入到 output 文件

以上代码加上了 `metaReader.IsAssembly` 用于判断当前的 DLL 是否 dotnet 程序集

以上代码的 PEReader 是 dotnet 默认提供的机制，在 Refasmer 库里面核心用是这个进行读取程序集内容

调用 MetadataImporter.MakeRefasm 方法的输出就是 byte 数组，可以写入到文件

上面代码的 VerySimpleLogger 是自己定义的日志类，代码忽略。可以从下面了解如何拿到本文源代码

本文所有代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/1b9b1a70f93fe065db216472d96a095eb7d39983/RairnarwayjallWhayderelaqea) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/1b9b1a70f93fe065db216472d96a095eb7d39983/RairnarwayjallWhayderelaqea) 欢迎访问

可以通过如下方式获取本文的源代码，先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin 1b9b1a70f93fe065db216472d96a095eb7d39983
```

以上使用的是 gitee 的源，如果 gitee 不能访问，请替换为 github 的源

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
```

获取代码之后，进入 RairnarwayjallWhayderelaqea 文件夹

更多关于 Roslyn 请看 [手把手教你写 Roslyn 修改编译](https://lindexi.oschina.io/lindexi/post/roslyn.html ) 

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://i.creativecommons.org/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
