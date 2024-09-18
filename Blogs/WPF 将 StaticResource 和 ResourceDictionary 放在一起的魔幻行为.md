---
title: WPF 将 StaticResource 和 ResourceDictionary 放在一起的魔幻行为
description: 本文将记录一些在 WPF 里面，使用 StaticResource 将 ResourceDictionary 玩坏的做法。大家可以放心的是，这些玩法基本只有高级玩家或逗比开发者才会使用到

<!--more-->

tags: WPF
category: 
---

<!-- CreateTime:2023/6/16 16:28:58 -->


<!-- 发布 -->
<!-- 博客 -->

## 后加入的资源无法被 StaticResource 找到

在 App.xaml.cs 后台代码里面，手动加入资源字典，手动加入的资源字典包含的资源，无法被提前在 App.xaml 加入的资源里面的 StaticResource 找到

测试方式如下

定义两个资源字典，分别是 Dictionary1.xaml 和 Dictionary2.xaml 字典。在 Dictionary1 里定义资源，在 Dictionary2 使用 StaticResource 引用 Dictionary1 的资源。在 App.xaml 引用 Dictionary2.xaml 字典，在 App.xaml.cs 加入 Dictionary1.xaml 字典。此时运行将会发现 Dictionary2 里使用 StaticResource 的属性的值是 DependencyProperty.UnsetValue 值，表示找不到资源

细节的步骤如下

定义两个资源字典，分别是 Dictionary1.xaml 和 Dictionary2.xaml 字典

在 Dictionary1.xaml 里面定义资源，如以下代码

```xml
<ResourceDictionary xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
                    xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml">
    <SolidColorBrush x:Key="SolidColorBrush" Color="Black"></SolidColorBrush>
</ResourceDictionary>
```

在 Dictionary2 使用 StaticResource 引用 Dictionary1 的资源，如以下代码

```xml
<ResourceDictionary xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
                    xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml">
    <Style x:Key="RectangleStyle" TargetType="Rectangle">
        <Setter Property="Fill" Value="{StaticResource SolidColorBrush}"></Setter>
    </Style>
</ResourceDictionary>
```

在 App.xaml 里只引用 Dictionary2.xaml 字典，如以下代码

```xml
<Application x:Class="JayabawwiWhenenearfajay.App"
             xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
             xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
             xmlns:local="clr-namespace:JayabawwiWhenenearfajay"
             StartupUri="MainWindow.xaml">
    <Application.Resources>
         <ResourceDictionary>
             <ResourceDictionary.MergedDictionaries>
                <!-- <ResourceDictionary Source="Dictionary1.xaml"></ResourceDictionary> -->
                <ResourceDictionary Source="Dictionary2.xaml"></ResourceDictionary>
             </ResourceDictionary.MergedDictionaries>
         </ResourceDictionary>
    </Application.Resources>
</Application>
```

在 App.xaml.cs 加入 Dictionary1.xaml 字典

```csharp
public partial class App : Application
{
    protected override void OnStartup(StartupEventArgs e)
    {
        var resourceDictionary = new ResourceDictionary()
        {
            Source = new Uri("/Dictionary1.xaml", UriKind.RelativeOrAbsolute)
        };

        Resources.MergedDictionaries.Add(resourceDictionary);

        base.OnStartup(e);
    }
}
```

接着在 MainWindow.xaml 使用 Dictionary2.xaml 定义的资源，如以下代码

```xml
<Window x:Class="JayabawwiWhenenearfajay.MainWindow"
        xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        xmlns:d="http://schemas.microsoft.com/expression/blend/2008"
        xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006"
        xmlns:local="clr-namespace:JayabawwiWhenenearfajay"
        mc:Ignorable="d"
        Title="MainWindow" Height="450" Width="800">
    <Grid>
        <Rectangle Style="{StaticResource RectangleStyle}"></Rectangle>
    </Grid>
</Window>
```

运行程序，将提示以下代码

```
System.InvalidOperationException:““{DependencyProperty.UnsetValue}”不是属性“Fill”的有效值。”
```

这就证明了定义在 Dictionary2.xaml 的 RectangleStyle 里的 Fill 属性找不到资源。也就是 `Setter Property="Fill" Value="{StaticResource SolidColorBrush}"` 这里的 StaticResource 无法找到定义在 Dictionary1.xaml 的资源

以上测试代码放在[github](https://github.com/lindexi/lindexi_gd/tree/abef940468bef7af6bd9ceed8566229aafda5016/JayabawwiWhenenearfajay) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/abef940468bef7af6bd9ceed8566229aafda5016/JayabawwiWhenenearfajay) 欢迎访问

可以通过如下方式获取本文的源代码，先创建一个名为 JayabawwiWhenenearfajay 的空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin abef940468bef7af6bd9ceed8566229aafda5016
```

以上使用的是 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin abef940468bef7af6bd9ceed8566229aafda5016
```

获取代码之后，进入 JayabawwiWhenenearfajay 文件夹

## 自定义 ResourceDictionary 资源可影响 StaticResource 寻找策略

以上的例子看起来还算正常，接下来来点魔幻的玩法

测试方式如下

在后台代码定义继承 ResourceDictionary 的类型，在此类型里面定义好和 Dictionary1.xaml 里的资源重名的资源，此时 Dictionary2.xaml 的 StaticResource 在运行将找对资源

也就是经过一番玩法，居然发现 StaticResource 又从 Dictionary1.xaml 里找对资源了

细节的步骤如下

在上一个例子的项目前提下，再新建一个名为 FooResourceDictionary 的类型，在构造函数添加上和 Dictionary1.xaml 里的资源重名的资源，代码如下

```csharp
public class FooResourceDictionary : ResourceDictionary
{
    public FooResourceDictionary()
    {
        Add("SolidColorBrush", this);
    }

    protected override void OnGettingValue(object key, ref object value, out bool canCache)
    {
        Debug.WriteLine(key);
        base.OnGettingValue(key, ref value, out canCache);
    }
}
```

以上代码在构造函数特别有趣的加入了 `"SolidColorBrush"` 资源，且设置资源的 Value 是 `this` 值。这就意味着如果 StaticResource 直接使用 FooResourceDictionary 里的 `"SolidColorBrush"` 资源，将拿到 FooResourceDictionary 类型的资源，完全无法转换为 Brush 类型，将会失败。然而实际上有趣的是最终 StaticResource 还是能找对资源

以上代码为了方便调试，也重写了 OnGettingValue 方法，这个方法是为了后文的另一个魔幻行为。不重写也不会影响当前的例子的行为

接着将这个自定义的 FooResourceDictionary 类型加入到 App.xaml 里面，必须放在 Dictionary2.xaml 之前，如以下代码

```xml
<Application x:Class="JayabawwiWhenenearfajay.App"
             xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
             xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
             xmlns:local="clr-namespace:JayabawwiWhenenearfajay"
             StartupUri="MainWindow.xaml">
    <Application.Resources>
         <ResourceDictionary>
             <ResourceDictionary.MergedDictionaries>
                <!-- <ResourceDictionary Source="Dictionary1.xaml"></ResourceDictionary> -->
                <local:FooResourceDictionary/>
                <ResourceDictionary Source="Dictionary2.xaml"></ResourceDictionary>
             </ResourceDictionary.MergedDictionaries>
         </ResourceDictionary>
    </Application.Resources>
</Application>
```

接着依然是在 App.xaml.cs 里面加入 Dictionary1.xaml 资源，代码和之前的完全相同，没有做任何改动。同样的 MainWindow.xaml 里面也没有做任何的改动

运行代码，可以看到这一次执行正常，静态资源寻找到了定义在 Dictionary1.xaml 的资源，不会受到在 FooResourceDictionary 定义的影响

以上测试代码放在[github](https://github.com/lindexi/lindexi_gd/tree/ac01fffe3908bcf5b69b459e1d3a6e50aa207b9c/JayabawwiWhenenearfajay) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/ac01fffe3908bcf5b69b459e1d3a6e50aa207b9c/JayabawwiWhenenearfajay) 欢迎访问

可以通过如下方式获取本文的源代码，先创建一个名为 JayabawwiWhenenearfajay 的空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin ac01fffe3908bcf5b69b459e1d3a6e50aa207b9c
```

以上使用的是 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin ac01fffe3908bcf5b69b459e1d3a6e50aa207b9c
```

获取代码之后，进入 JayabawwiWhenenearfajay 文件夹

<!-- ## 使用 StaticResource 寻找资源不会进入 OnGettingValue 方法

使用上一个例子的代码完全不用做更改，只需要在 FooResourceDictionary 的 OnGettingValue 方法上打上断点即可。运行代码，可以看到断点一点都不会进入。这就证明了在 StaticResource 在寻找资源是不会进入 OnGettingValue 方法

当然了，大家要是担心是被 Dictionary1.xaml 覆盖的话，也可以删掉。然后删掉之后，居然就进了 -->

通过以上的两个例子可以说明 StaticResource 的行为是在资源加载的过程中就会执行，执行时将会尝试从资源字典里寻找静态资源 Key 的定义，如果有找到 Key 的记录，则加入延迟初始化逻辑。延迟初始化逻辑还没有绑定到具体哪个资源字典，是在实际需要获取值的时候，才进行重新确定实际的资源。这也就是为什么 FooResourceDictionary 的 OnGettingValue 方法没有进入的原因，因为 StaticResource 实际获取值是从 Dictionary1.xaml 获取的，完全不在 FooResourceDictionary 里获取

如果没有找到 Key 的记录，那就直接给属性赋值为 DependencyProperty.UnsetValue 属性，结束寻找。即使后续加入的资源字典添加了对应的资源，也不会重新更新。这个行为符合微软的文档，试试看交换两个有依赖关系的资源字典加入 App.xaml 的顺序，可以看到顺序倒了之后将导致静态资源找不到。这个行为和资源字典加入顺序导致的找不到资源是相同的

在此例子里面是通过在 FooResourceDictionary 的构造里面，构建了 `"SolidColorBrush"` 资源，从而让 StaticResource 静态绑定资源引用设置给属性一个延迟初始化值，在实际的界面使用时，获取到 Dictionary1.xaml 覆盖 FooResourceDictionary 的资源

有些资源如果想要延迟加入到 App.xaml 里面，延迟初始化资源字典的话，就需要考虑 StaticResource 寻找资源的问题。一个可选的方式是自己定义继承 ResourceDictionary 的类型，如本文的 FooResourceDictionary 类型，在类型的构造函数里面写满了 StaticResource 可能使用的资源，从而让 StaticResource 加入延迟初始化逻辑

## 在后台代码加入新资源字典之前读取静态资源引用的值

上一个例子可以正确获取到资源，在上一个例子的基础上，后台代码加入 Dictionary1.xaml 之前，尝试获取 StaticResource 静态绑定资源引用的值。获取到的值，可以看到获取到的是定义在 FooResourceDictionary 里的资源，很符合预期。但有趣的是，之后尽管加入了 Dictionary1.xaml 但静态资源引用的值不会更新，应用无法跑起来，将提示以下代码

```
System.InvalidOperationException:““JayabawwiWhenenearfajay.FooResourceDictionary”不是属性“Fill”的有效值。”
```

详细的步骤如下

只在 App.xaml.cs 的 Dictionary1.xaml 加入之前，添加以下代码用来获取静态绑定资源引用属性的值

```csharp
        var value = ((System.Windows.Setter) (Resources.MergedDictionaries)[1].Values.OfType<Style>().First().Setters[0]).Value;
```

修改之后的代码如下

```csharp
public partial class App : Application
{
    protected override void OnStartup(StartupEventArgs e)
    {
        // 注释这句话试试
        var value = ((System.Windows.Setter) (Resources.MergedDictionaries)[1].Values.OfType<Style>().First().Setters[0]).Value;

        var resourceDictionary = new ResourceDictionary()
        {
            Source = new Uri("/Dictionary1.xaml", UriKind.RelativeOrAbsolute)
        };

        Resources.MergedDictionaries.Add(resourceDictionary);

        base.OnStartup(e);
    }
}
```

这里可以看到 value 获取时，将进入 FooResourceDictionary 的 OnGettingValue 函数。拿到的 value 是 FooResourceDictionary 类型，也就是这个资源是在 FooResourceDictionary 提供的。符合预期，因为此时 Dictionary1.xaml 还没加入

但有趣的是在应用运行的时候，即使 Dictionary1.xaml 已经加入，此时拿到的还是原来的 FooResourceDictionary 类型，从而运行失败

这个行为不算魔幻，这是因为 StaticResource 只执行一次，即使后续的字典变更了，也不会重新执行。这是 StaticResource 和 DynamicResource 的差别，这也就是使用 StaticResource 时性能更高的原因。以上的代码在 Dictionary1.xaml 加入之前，获取 StaticResource 静态资源引用绑定的属性的值，从而让 StaticResource 执行，找到了在 FooResourceDictionary 定义的资源。由于 StaticResource 只执行一次，这就导致了即使后续加入 Dictionary1.xaml 资源字典，也不会更新 StaticResource 静态资源引用绑定的属性的值为 Dictionary1.xaml 资源字典的资源，于是应用程序就拿到了错误的对象放入 Fill 属性，运行失败

以上测试代码放在[github](https://github.com/lindexi/lindexi_gd/tree/cf93266c7077a9b4acea939ce198bd7a8abe6536/JayabawwiWhenenearfajay) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/cf93266c7077a9b4acea939ce198bd7a8abe6536/JayabawwiWhenenearfajay) 欢迎访问

可以通过如下方式获取本文的源代码，先创建一个名为 JayabawwiWhenenearfajay 的空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin cf93266c7077a9b4acea939ce198bd7a8abe6536
```

以上使用的是 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin cf93266c7077a9b4acea939ce198bd7a8abe6536
```

获取代码之后，进入 JayabawwiWhenenearfajay 文件夹


## 资源字典树引用与资源寻找的坑

详细请参阅 [WPF 已知问题 资源字典树引用与资源寻找的坑](https://blog.lindexi.com/post/WPF-%E5%B7%B2%E7%9F%A5%E9%97%AE%E9%A2%98-%E8%B5%84%E6%BA%90%E5%AD%97%E5%85%B8%E6%A0%91%E5%BC%95%E7%94%A8%E4%B8%8E%E8%B5%84%E6%BA%90%E5%AF%BB%E6%89%BE%E7%9A%84%E5%9D%91.html )
