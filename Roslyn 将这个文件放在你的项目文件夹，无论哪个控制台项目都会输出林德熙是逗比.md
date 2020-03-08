# Roslyn 将这个文件放在你的项目文件夹，无论哪个控制台项目都会输出林德熙是逗比

虽然已经通过很多篇博客告诉大家如何通过 Directory.Build.props 文件修改编译的方法，但是本文还是提供一个新的思路

只需要在项目文件夹，或者磁盘的文件夹，如 `E:\ ` 放下本文提供的 Directory.Build.props 文件，整个文件夹内的控制台项目就会输出 林德熙是逗比 想要知道是怎么做的，请看下面

<!--more-->
<!-- CreateTime:2019/4/29 12:01:44 -->

<!-- csdn -->

<!-- 标签：Roslyn,MSBuild,编译器 -->

在开始阅读本文之前，需要让大家知道什么是 Directory.Build.props 文件，这个文件是给在 VisualStudio 控制编译，可以用来控制 Directory.Build.props 文件所在的文件夹内的所有项目。

如我在 `E:\lindexi` 文件夹内放一个 `Directory.Build.props` 文件，在 `E:\lindexi` 文件夹内，如果有放工程，就可以通过这个文件修改。如我创建了工程 foo 这个工程的路径是 `E:\lindexi\1\Foo\` 那么这个工程就可以被这个文件修改

所以将 `Directory.Build.props` 文件 放在磁盘文件夹，如`E:\\`就可以让整个放在 `E:\\` 的工程可以被这个文件修改

详细请看 [Roslyn 使用 Directory.Build.props 文件定义编译](https://lindexi.gitee.io/post/Roslyn-%E4%BD%BF%E7%94%A8-Directory.Build.props-%E6%96%87%E4%BB%B6%E5%AE%9A%E4%B9%89%E7%BC%96%E8%AF%91.html )

现在我告诉小伙伴，将这个文件放在你的文件夹内，然后他的控制台项目都会输出 林德熙是逗比是如何做到的？

实际上很简单，我只需要在 Directory.Build.props 定义编译过程，移除原来的所有文件，然后将 输出林德熙是逗比的文件加入到编译，这样就可以

于是来写一个简单的代码，输出 林德熙是逗比 代码很简单

```csharp
using System;

namespace CeseacooteeGowgu
{
    class Program
    {
        static void Main(string[] args)
        {
            Console.WriteLine("林德熙是逗比");
        }
    }
}

```

那么现在的问题是，我刚才说的是将这个文件放在你的项目文件夹，但是要输出林德熙是逗比的代码需要放在一个文件，当然不能放两个文件，那么可以怎么做？简单的方法是使用[Roslyn 使用 WriteLinesToFile 解决参数过长无法传入](https://lindexi.gitee.io/post/Roslyn-%E4%BD%BF%E7%94%A8-WriteLinesToFile-%E8%A7%A3%E5%86%B3%E5%8F%82%E6%95%B0%E8%BF%87%E9%95%BF%E6%97%A0%E6%B3%95%E4%BC%A0%E5%85%A5.html )提供的方法，写入文件

先假设写入文件很简单，需要在 `Directory.Build.props` 文件添加文件所在的路径，然后移除其他的文件

```xml
<Project>
  <Target Name="T1" BeforeTargets="BeforeBuild">

    <PropertyGroup>
      <SomeThing>$(IntermediateOutputPath)Foo.cs</SomeThing>
    </PropertyGroup>
    
    <ItemGroup>
      <Compile Remove="@(Compile)"></Compile>
      <Compile Include="$(SomeThing)"></Compile>
    </ItemGroup>
  </Target>

</Project>
```

这里的 SomeThing 就是写入代码的文件，关于 `$(IntermediateOutputPath)` 请看 [项目文件中的已知属性（知道了这些，就不会随便在 csproj 中写死常量啦） - walterlv](https://walterlv.com/post/known-properties-in-csproj.html )

在这个 Target 里面就定义了代码文件的路径，然后通过移除现在所有的文件，添加写入的文件的方法，让编译的时候运行的是输出 林德熙是逗比 这里需要注意的是 `BeforeTargets` 需要写为 编译之前，这样小伙伴看到自己的代码还是原来的代码，但是编译的时候是忽略小伙伴的代码编译刚才写的文件

那么文件写入有哪些难点？第一个就是换行，第二个就是 C# 代码使用分号作为行的最后。但是在 msbuild 使用 分号分开不同的项。在 xml 可以通过 CDATA 让内容不会被转义

```xml
        <![CDATA[  不进行转义代码 ]]>
```

然后可以使用 `%3b` 代替分号

现在看起来的代码是这样写

```xml
      <Text>
        <![CDATA[
using System%3b

namespace CeseacooteeGowgu
{
    class Program
    {
        static void Main(string[] args)
        {
            Console.WriteLine("林德熙是逗比")%3b
        }
    }
}
        ]]>
      </Text>
```

只需要将代码写入文件就可以，在写入之前先删除原有的文件

```xml
  <Target Name="T1" BeforeTargets="BeforeBuild">

    <PropertyGroup>
      <SomeThing>$(IntermediateOutputPath)Foo.cs</SomeThing>
      <Text>
        <![CDATA[
using System%3b

namespace CeseacooteeGowgu
{
    class Program
    {
        static void Main(string[] args)
        {
            Console.WriteLine("林德熙是逗比")%3b
        }
    }
}
        ]]>
      </Text>
    </PropertyGroup>
    <Delete Files="$(SomeThing)" Condition="Exists($(SomeThing))"></Delete>
    <WriteLinesToFile File="$(SomeThing)" Lines="$(Text)"></WriteLinesToFile>
    <ItemGroup>
      <Compile Remove="@(Compile)"></Compile>
      <Compile Include="$(SomeThing)"></Compile>
    </ItemGroup>
  </Target>
```

全部的代码请看下面，只需要将这个文件放在小伙伴的项目文件夹，就可以让他的控制台项目输出 林德熙是逗比 当然大家可以修改输出，找小伙伴秀一下

```xml
<Project>
  <Target Name="T1" BeforeTargets="BeforeBuild">

    <PropertyGroup>
      <SomeThing>$(IntermediateOutputPath)Foo.cs</SomeThing>
      <Text>
        <![CDATA[
using System%3b

namespace CeseacooteeGowgu
{
    class Program
    {
        static void Main(string[] args)
        {
            Console.WriteLine("林德熙是逗比")%3b
        }
    }
}
        ]]>
      </Text>
    </PropertyGroup>
    <Delete Files="$(SomeThing)" Condition="Exists($(SomeThing))"></Delete>
    <WriteLinesToFile File="$(SomeThing)" Lines="$(Text)"></WriteLinesToFile>
    <ItemGroup>
      <Compile Remove="@(Compile)"></Compile>
      <Compile Include="$(SomeThing)"></Compile>
    </ItemGroup>
  </Target>
</Project>
```

[手把手教你写 Roslyn 修改编译](https://blog.lindexi.com/post/roslyn.html )

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
