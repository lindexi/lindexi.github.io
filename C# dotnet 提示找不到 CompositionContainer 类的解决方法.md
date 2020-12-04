# C# dotnet 提示找不到 CompositionContainer 类的解决方法

在构建提示 Error CS0012 和 Error CS0246 说找不到 CompositionContainer 类，原因是没有引用 System.ComponentModel.Composition 库

<!--more-->
<!-- CreateTime:6/17/2020 4:01:19 PM -->



在构建的时候有如下提示

```
0>MainWindow.xaml.cs(51,59): Error CS0246: The type or namespace name 'CompositionContainer' could not be found (are you missing a using directive or an assembly reference?)
0>MainWindow.xaml.cs(51,35): Error CS0012: The type 'CompositionContainer' is defined in an assembly that is not referenced. You must add a reference to assembly 'System.ComponentModel.Composition, Version=4.0.0.0, Culture=neutral, PublicKeyToken=b77a5c561934e089'.
```

原因是没有引用 System.ComponentModel.Composition 库，双击项目编辑 csproj 项目，添加下面代码

```xml
    <ItemGroup>
        <Reference Include="System.ComponentModel.Composition"/>
    </ItemGroup>
```

要求 csproj 是 sdk 风格的，如果不是，请右击引用，添加引用，找到 System.ComponentModel.Composition 勾选

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
