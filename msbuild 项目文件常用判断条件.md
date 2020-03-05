# msbuild 项目文件常用判断条件

在写项目文件的时候，需要根据不同的条件定义或执行不同的代码，有一些比较常使用的判断，本文收藏起来，方便大家找

<!--more-->
<!-- CreateTime:2019/11/29 8:36:48 -->

<!-- 标签：Roslyn,MSBuild,编译器 -->


在 msbuild 的项目文件 cspoj 或 xx.target 等文件里面，可以使用 Condition 条件写在很多标签元素作为判断

例如在 Target 上面添加条件，只有条件满足了才会执行

```csharp
  <Target Name="Lindexi" AfterTargets="CoreCompile" Condition="'$(Configuration)|$(TargetFramework)'=='DEBUG|net45'">
    <Message Text="林德熙是逗比"></Message>
  </Target>
```

下面将告诉大家一些常使用的判断

## 判断在调试下编译

请看代码

```csharp
Condition="'$(Configuration)'=='Debug'"
```

这里 Configuration 的判断是不区分大小写的，默认写的是 `Debug` 而 `DEBUG` 是用在条件编译里面

例如这样写

```csharp
  <PropertyGroup Condition=" '$(Configuration)' == 'Debug'">
      <MainProjectPath>blog.lindexi.com</MainProjectPath>
  </PropertyGroup>
```

## 判断在发布下编译

请看代码

```csharp
Condition="'$(Configuration)'!='Debug'"
```

也就是上面代码反过来判断不是在调试下编译

另一个判断方法请看代码，这是不推荐的判断方法

```csharp
Condition="'$(Configuration)'=='Release'"
```

这个不推荐的写法，一般只有调试下和非调试下，用上面的写法可能有逗比写了 `Release-x` 于是就判断不是发布下，此时就没有做发布的优化

## 判断平台

判断在 .NET Framework 4.5 运行

```csharp
Condition="'$(TargetFramework)'=='net45'"
```

对应的判断 .NET Standard 使用如下缩写 `netstandard1.0` 等

判断 .NET Core 使用如下缩写 `netcoreapp1.0` 等

## 多个判断

需要同时生效有两个写法，如判断只有在 .NET Framework 4.5 同时在调试下

```csharp
Condition="'$(Configuration)|$(TargetFramework)'=='DEBUG|net45'"
```

第二个方法是使用关键字 And 连接

```csharp
Condition=" '$(TargetFramework)'=='net45' And $(Configuration)=='Debug'"
```

两个条件的或判断使用关键字 Or 连接

```csharp
Condition=" '$(TargetFramework)'=='net45' or $(Configuration)=='Debug'"
```

更多判断请看

[Roslyn 在项目文件使用条件判断](https://blog.lindexi.com/post/Roslyn-%E5%9C%A8%E9%A1%B9%E7%9B%AE%E6%96%87%E4%BB%B6%E4%BD%BF%E7%94%A8%E6%9D%A1%E4%BB%B6%E5%88%A4%E6%96%AD.html )

[MSBuild 如何编写带条件的属性、集合和任务 Condition？ - walterlv](https://blog.walterlv.com/post/how-to-write-msbuild-conditions.html )

[Target frameworks](https://docs.microsoft.com/en-us/dotnet/standard/frameworks?wt.mc_id=MVP )

[手把手教你写 Roslyn 修改编译](https://blog.lindexi.com/post/roslyn.html )

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
