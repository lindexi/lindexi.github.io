# VisualStudio 2019 尝试使用 C# 8.0 新的方式

在安装了几天之后，终于有了 VisualStudio 2019 于是再安装了 dotnet core 3.0 预览版，现在可以来尝试使用 C# 8.0 的新方式

<!--more-->
<!-- csdn -->

新的 VisualStudio 界面十分清真，此时可以通过标签找到自己可以创建的项目

<!-- ![](image/VisualStudio 2019 尝试使用 C# 80 新的方式/VisualStudio 2019 尝试使用 C# 80 新的方式1.png) -->

![](http://image.acmx.xyz/lindexi%2F20181212174618459)

找到一个简单的控制台项目，创建的界面也很简单

<!-- ![](image/VisualStudio 2019 尝试使用 C# 80 新的方式/VisualStudio 2019 尝试使用 C# 80 新的方式0.png) -->

![](http://image.acmx.xyz/lindexi%2F20181212174552691)

虽然界面不错，但是创建项目的速度没有加快，等了很久，终于看到和之前没有多少修改的界面

在按下调试的时候，发现有一些按钮的界面修改，如下一步的按钮

<!-- ![](image/VisualStudio 2019 尝试使用 C# 80 新的方式/VisualStudio 2019 尝试使用 C# 80 新的方式2.png) -->

![](http://image.acmx.xyz/lindexi%2F201812121753079)

当前，在使用之前需要做一些准备，首先是[下载](https://aka.ms/netcore3download) dotnet core 3.0 不然一些功能不能使用

右击项目，编辑一下 csproj 文件

```
 <Project Sdk="Microsoft.NET.Sdk">

  <PropertyGroup>
    <OutputType>Exe</OutputType>
    <TargetFramework>netcoreapp3.0</TargetFramework>
  </PropertyGroup>

</Project>
```

右击项目属性，在生成页面选择用最新语言

<!-- ![](image/VisualStudio 2019 尝试使用 C# 80 新的方式/VisualStudio 2019 尝试使用 C# 80 新的方式3.png) -->

![](http://image.acmx.xyz/lindexi%2F2018121221123464)

## 可空类型

现在可以提示开发者写出可能为空的代码，如 string 默认可以设置为不可空

```csharp
        static void Main(string[] args)
        {
            string str = null;
            Console.WriteLine(str);
        }
```

以前这样写代码是可以的，但是现在，可以在 csproj 文件里面添加 NullableReferenceTypes 此时就会在设置 string 为空提示

现在的 csproj 需要添加 NullableReferenceTypes 请看下面

```
<Project Sdk="Microsoft.NET.Sdk">

  <PropertyGroup>
    <OutputType>Exe</OutputType>
    <TargetFramework>netcoreapp3.0</TargetFramework>
    <LangVersion>8.0</LangVersion>
    <NullableReferenceTypes>true</NullableReferenceTypes>
  </PropertyGroup>

</Project>
```

再次编译一下项目，虽然可以看到项目编译通过，但是可以看到下面的警告

<!-- ![](image/VisualStudio 2019 尝试使用 C# 80 新的方式/VisualStudio 2019 尝试使用 C# 80 新的方式4.png) -->

![](http://image.acmx.xyz/lindexi%2F20181212211819918)

现在对于 string 是不可空的，但是为了兼容以前的代码，没有强制让编译不通过。同时提供了可空的字符串，也就是 `string?` 请看代码

```csharp
            string? str = null;
            Console.WriteLine(str);
```

那么原来的 `str[0]` 的这些写法需要怎么修改呢？

答案是可以不修改，直接写

```csharp
            string? str = null;
            var foo = str[0];
            Console.WriteLine(str);
```

但是编译的时候会出现下面的警告

```csharp
warning CS8602: Possible dereference of a null reference.
```

如果需要在 string 判断为空的时候不使用，可以使用下面的方法

```csharp
            string? str = null;
            var foo = str?[0];
            Console.WriteLine(str);
```

## Range

第二个好玩的是 Range 可以指定使用数组的哪些内容

```csharp
            var foo = new[] { "1 lindexi", "2 doubi", "3 csdn" };

            foreach(var temp in foo[0..1])
            {
                Console.WriteLine(temp);
            }
```

此时输出从第 0 元素到第 1 元素，不包括第 1 元素的值

```csharp
1 lindexi
```

如果修改为 `foo[0..2]` 就会输出 

```csharp
1 lindexi
2 doubi
```

如果需要从第 1 个元素到最后一个元素，可以这样写

```csharp
            var foo = new[] { "1 lindexi", "2 doubi", "3 csdn" };

            foreach(var temp in foo[1..])
            {
                Console.WriteLine(temp);
            }

            // 2 doubi
            // 3 csdn
```

如果想要从第 0 个元素输出到倒数第一个元素，不包括倒数第一个元素，可以这样写

```csharp
            var foo = new[] { "1 lindexi", "2 doubi", "3 csdn" };

            foreach(var temp in foo[1..^1])
            {
                Console.WriteLine(temp);
            }

            // 2 doubi
```

当然 `1..2` 这个只是一个语法糖，这是一个 Range 类

```csharp
            Range range = 1..2;

            foreach (var temp in foo[range])
            {
                Console.WriteLine(temp);
            }
```

关于 Range 请看[Range Type in C# 8 - .NET Core Tutorials](https://dotnetcoretutorials.com/2018/12/09/range-type-in-c-8/ )

[C# 8中的范围类型(Range Type) - LamondLu - 博客园](https://www.cnblogs.com/lwqlun/p/10095821.html )

## 异步的流

可以通过 async 修饰 yield 返回的方法

```csharp
            await foreach (var temp in Foo())
            {
                Console.WriteLine(temp);
            }

            async IAsyncEnumerable<string> Foo()
            {
                foreach(var temp in new[] { "1 lindexi", "2 doubi", "3 csdn" })
                {
                    await Task.Delay(100);
                    yield return temp;
                }
            }
```

这样就可以异步返回

听说安装了 VisualStudio 预览版就无法安装正式版，除非重装系统，所以小伙伴要不要也来弄个预览版玩一下，点击[安装](https://visualstudio.microsoft.com/vs/preview/)预览版

更多请看 [Building C# 8.0](https://blogs.msdn.microsoft.com/dotnet/2018/11/12/building-c-8-0/ )

参见：[Take C# 8.0 for a spin](https://blogs.msdn.microsoft.com/dotnet/2018/12/05/take-c-8-0-for-a-spin/ )

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
