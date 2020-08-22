
# dotnet 新 SDK Style 项目格式如何使用 InternalsVisibleToAttribute 功能

如果一个项目想要让其他某个指定的项目可以使用到 internal 的类或成员，可以通过标记 InternalsVisibleToAttribute 的方式实现

<!--more-->


<!-- 发布 -->

最简单的方法是新建一个 AssemblyInfo.cs 文件，在这个文件里面使用 System.Runtime.CompilerServices.InternalsVisibleToAttribute 指定某个程序集可见

```csharp
using System;
using System.Reflection;

[assembly: System.Runtime.CompilerServices.InternalsVisibleToAttribute("Lindexi.blog.csdn.net")]
```

对于强签名的程序集，只能被强签名的程序集可见

如果想要在 csproj 文件上面写，也可以，在 ItemGroup 添加下面代码

```xml
    <ItemGroup>
        <AssemblyAttribute Include="System.Runtime.CompilerServices.InternalsVisibleToAttribute">
            <_Parameter1>Lindexi.blog.csdn.net</_Parameter1>
        </AssemblyAttribute>
    </ItemGroup>
```

为什么这样就可以实现和原先 AssemblyInfo.cs 相同的功能？其实在构建的之前，将会执行预编译，将 AssemblyAttribute 的内容输出到 `obj\*.AssemblyInfo.cs` 文件，小伙伴可以尝试打开这个文件，其实这个文件是由 WriteCodeFragment 生成，内容和刚才的 AssemblyInfo.cs 文件是差不多的

所以本质上是通过预编译创建 AssemblyInfo.cs 文件实现。只是用这个方法可以不手工创建 AssemblyInfo.cs 文件

如我创建的 WPF 项目，这个项目里面有一个 Foo 类，期望被其他两个项目使用，此时可以添加如下代码

```xml
    <ItemGroup>
        <AssemblyAttribute Include="System.Runtime.CompilerServices.InternalsVisibleToAttribute">
            <_Parameter1>LeerijawarnikoJacallnachaykall</_Parameter1>
        </AssemblyAttribute>
        <AssemblyAttribute Include="System.Runtime.CompilerServices.InternalsVisibleToAttribute">
            <_Parameter1>HikallyijuDallcurjemdehowai</_Parameter1>
        </AssemblyAttribute>
    </ItemGroup>
```

代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/58fd8f2d6f20c57f13fb216f5b9872778801cabc/JikaniqayfaraineWaycarjeefer) 欢迎小伙伴访问

当然这么写代码比较乱，可以通过小伙伴 [Meziantou](https://www.meziantou.net ) 的方法，只需要安装一个有趣的 NuGet 包，就可以使用十分清真的写法。安装 NuGet 的方法是在 csproj 里面的添加下面代码

```xml
    <ItemGroup>
        <PackageReference Include="Meziantou.MSBuild.InternalsVisibleTo" Version="1.0.2">
            <PrivateAssets>all</PrivateAssets>
            <IncludeAssets>runtime; build; native; contentfiles; analyzers</IncludeAssets>
        </PackageReference>
    </ItemGroup>
```

在安装完成了 Meziantou.MSBuild.InternalsVisibleTo 库之后，可以使用下面代码让其他项目可见 internal 的类

```xml
    <ItemGroup>
        <InternalsVisibleTo Include="LeerijawarnikoJacallnachaykall"/>
        <InternalsVisibleTo Include="HikallyijuDallcurjemdehowai"/>
    </ItemGroup>
```

那么这个 NuGet 库的原理是什么？其实是通过 [通过重写预定义的 Target 来扩展 MSBuild / Visual Studio 的编译过程 - walterlv](https://blog.walterlv.com/post/extend-the-visual-studio-build-process.html ) 的方法，类似如下代码实现

```xml
  <Target Name="AddInternalsVisibleTo" BeforeTargets="BeforeCompile">
    <ItemGroup Condition="'@(InternalsVisibleTo->Count())' &gt; 0">
      <AssemblyAttribute Include="System.Runtime.CompilerServices.InternalsVisibleTo">
        <_Parameter1>%(InternalsVisibleTo.Identity)</_Parameter1>
      </AssemblyAttribute>
    </ItemGroup>
  </Target>
```

也就是添加一个 AddInternalsVisibleTo 的 Target 在开始构建之前触发

内容就是读取 InternalsVisibleTo 项的内容，加入到 AssemblyAttribute 这里，而在上文小伙伴了解到放在 AssemblyAttribute 的内容将会被输出

用这个方法就可以做到写 InternalsVisibleTo 就可以自动创建 AssemblyInfo.cs 文件







<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。