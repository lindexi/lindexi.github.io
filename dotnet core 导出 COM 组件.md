# dotnet core 导出 COM 组件

在 dotnet core 3.0 支持将库导出为COM组件，本文告诉大家如何将代码导出为 COM 组件

<!--more-->
<!-- CreateTime:2019/9/22 20:25:38 -->

<!-- csdn -->

在导出 COM 组件的库，需要一个 GUID 声明这个 COM 接口

例如创建一个项目，在这个项目添加一个接口，通过以下方法标记为 COM 组件

这里的 Guid 是我自己设置的，可以在 VisualStudio 工具里面找到 GUID 创建选项，创建一个随机的 GUID 字符串

```csharp
    [ComVisible(true)]
    [Guid("5742D257-CCCC-4F7A-8191-6362609C458D")]
    [InterfaceType(ComInterfaceType.InterfaceIsIUnknown)]
    public interface IFoo
    {
        /// <summary>
        /// 有趣方法
        /// </summary>
        /// <returns></returns>
        string Foo();
    }

```

上面代码就定义了一个接口，在相同的项目还需要写一个类实现这个接口

```csharp
    [ComVisible(true)]
    [Guid("5742D257-CCCC-4F7A-8191-6362609C458D")]
    public class Foo : IFoo
    {
        /// <inheritdoc />
        string IFoo.Foo()
        {
            return "林德熙是逗比";
        }
    }
```

编辑这个项目的 csproj 添加属性 EnableComHosting 这样编译的时候才会生成可以导出为COM的文件

```csharp
  <PropertyGroup>
    <EnableComHosting>true</EnableComHosting>
  </PropertyGroup>
```

现在看起来的项目文件如下

```csharp
<Project Sdk="Microsoft.NET.Sdk">

  <PropertyGroup>
    <OutputType>Library</OutputType>
    <TargetFramework>netcoreapp3.0</TargetFramework>
  </PropertyGroup>

  <PropertyGroup>
    <EnableComHosting>true</EnableComHosting>
  </PropertyGroup>
</Project>
```

现在尝试编译这个项目，可以在输出路径里面找到 项目名.comhost.dll 文件，如创建的项目是 BearqalkeawaiKaleenemcemfo 那么在输出文件夹可以找到这个文件

```csharp
BearqalkeawaiKaleenemcemfo.comhost.dll
```

使用管理员运行命令行，输入下面代码注册 COM 文件

```csharp
regsvr32 项目名.comhost.dll
```

尝试创建一个新项目引用这个COM组件，使用的方法是定义一个接口

```csharp
    [ComImport]
    [CoClass(typeof(Foo))]
    [Guid("5742D257-CCCC-4F7A-8191-6362609C458D")]
    public interface IFoo
    {
        /// <summary>
        /// 有趣方法
        /// </summary>
        /// <returns></returns>
        string Foo();
    }

    [ComImport]
    [Guid("5742D257-CCCC-4F7A-8191-6362609C458D")]
    internal class Foo
    {
    }
```

这里定义的方法和其他使用COM的方法相同

```csharp
        static void Main(string[] args)
        {
            var foo = new IFoo();
            Console.WriteLine(foo.Foo());
        }
```

创建接口就可以使用方法

[Exposing .NET Core Components to COM](https://docs.microsoft.com/en-us/dotnet/core/native-interop/expose-components-to-com )

源代码请看 [github](https://github.com/lindexi/lindexi_gd/tree/de3c493051f15be07c4327e797d081c6869c6f93/BearqalkeawaiKaleenemcemfo) 和 [官方源代码](https://github.com/dotnet/samples/tree/master/core/extensions/COMServerDemo) 

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
