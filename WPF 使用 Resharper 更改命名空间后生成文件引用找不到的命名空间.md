# WPF 使用 Resharper 更改命名空间后生成文件引用找不到的命名空间

在 Resharper 更改全部命名空间之后，在 xx.g.cs 文件里面的 using 用了一个之前的命名空间，但是代码里面没有地方使用，此时构建不通过，原因是 xaml 里面存在引用

<!--more-->
<!-- CreateTime:6/18/2020 5:52:59 PM -->



在安装 Resharper 之后，可以右击某个文件夹或项目，点击 Refactor -> AdjustNamespaces 批量更改命名空间

尽管 Resharper 会将大量的 xaml 的元素改对了命名空间，但是有些没有用到的标签就没有改全

如下面代码

```xml
<Window x:Class="Lindexi.Doubi.MainWindow"
        xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        xmlns:d="http://schemas.microsoft.com/expression/blend/2008"
        xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006"
        xmlns:local="clr-namespace:Lindexi.DoubiDemo"
        mc:Ignorable="d"
        ResizeMode="NoResize"
        Title="MainWindow" Height="450" Width="800">
    <Grid>
        <Border Background="Gray" Width="100" Height="100" Margin="10,10,10,10" >
            <Button Click="Button_OnClick">点击</Button>
        </Border>
    </Grid>
</Window>

```

这里的 `local="clr-namespace:Lindexi.DoubiDemo"` 没有用到，我将命名空间从 `Lindexi` 修改为 `Lindexi.Doubi` 但是这里没有更改，而在生成的 MainWindow.g.cs 文件会根据 xaml 的命名空间引用添加 `using` 代码

因此在 g.cs 文件会创建下面代码

```csharp
using Lindexi.DoubiDemo;
```

构建的时候提示找不到类或命名空间

```
>obj\Debug\net45\MainWindow.g.cs(12,18): Error CS0234: The type or namespace name 'DoubiDemo' does not exist in the namespace 'Lindexi' (are you missing an assembly reference?)
```

解决方法有两个

第一个解决方法就是删除 xaml 里面的这些引用，可以全局搜寻，替换字符串

第二个方法是自己写一个空白的命名空间，因为如果 xaml 太多的话，需要改很久，写一个空白的命名空间就可以

```csharp
namespace Lindexi.DoubiDemo
{

}
```

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
