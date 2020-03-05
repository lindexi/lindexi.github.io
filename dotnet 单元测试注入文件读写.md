# dotnet 单元测试注入文件读写

在进行文件读写时，如果进行单元测试，是需要很多设计，因为本地的文件可能因为单元测试之间的占用，以及还原数据，会影响业务。本文告诉大家使用注入的方式，让文件读写一个抽象的对象

<!--more-->
<!-- CreateTime:2019/12/10 19:44:16 -->

<!-- 发布 -->

单元测试文件读写的难点是构建出测试的文件，如要求文件的权限是用户不可读，如要求文件的长度很长，如要求文件的修改时间。而 System.IO.Abstractions 这个库提供了注入抽象的文件，所有属性都可以设置

首先安装 System.IO.Abstractions 库，这个库提供了 IFileSystem 接口，使用这个接口提供的文件读写方法代替静态类 File 等方法。这样通过注入 IFileSystem 接口，可以进行文件读写测试

```csharp
Install-Package System.IO.Abstractions
```

在使用 System.IO.Abstractions 库进行注入，需要对代码进行一些修改，如将 `File.ReadAllText` 的代码修改为 `fileSystem.File.ReadAllText` 这里的 fileSystem 是 IFileSystem 进行注入

在进行单元测试的注入，可选的是构造注入等方法，通过注入的 IFileSystem 属性进行文件读写

```csharp
    public class Foo
    {
        /// <inheritdoc />
        public Foo(IFileSystem fileSystem)
        {
            FileSystem = fileSystem;
        }

        public IFileSystem FileSystem { get; }
    }
```

之后的文件读写，大概修改如 File.ReadAllText 和 FileStream 代码

```csharp
            IFileSystem fileSystem = FileSystem;

            File.ReadAllText("lindexi.txt");
            fileSystem.File.ReadAllText("lindexi.txt");
```

也就是将 File.ReadAllText 替换 fileSystem.File.ReadAllText 方法

```csharp
            new FileStream("blog.lindexi.com", FileMode.Create);
            fileSystem.FileStream.Create("blog.lindexi.com", FileMode.Create);
```

此时进行单元测试注入，这里用 Mock 的方法创建一个抽象的对象

单元测试需要按照 System.IO.Abstractions.TestingHelpers 库

注入 IFileSystem 方法

```csharp
            var mockFileSystem = new MockFileSystem(new Dictionary<string, MockFileData>()
            {
                {
                    "文件路径", new MockFileData("文件内容")
                },
                { "文件夹", new MockDirectoryData() }
            });

            var foo = new Foo(mockFileSystem);
```

在 MockFileSystem 支持设置文件或文件夹的属性，不需要存在实际的文件

如果需要虚拟更多接口，请使用 Mock 方法

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://i.creativecommons.org/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
