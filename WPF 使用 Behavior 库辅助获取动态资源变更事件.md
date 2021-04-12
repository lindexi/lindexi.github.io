# WPF 使用 Behavior 库辅助获取动态资源变更事件

在 WPF 开发中，可以使用 Behavior 库辅助，监听某个动态资源变更的事件，从而了解到是在哪个模块变更了动态资源，或者根据动态资源的变更而进行界面修改

<!--more-->
<!-- 发布 -->

在 WPF 的动态资源机制里面，如果某个依赖属性给定了动态资源，那么在动态资源变更的时候，将会重新给属性赋值。通过这个机制，就可以在业务逻辑上，通过添加一个依赖属性，绑定到需要监听变更的动态资源上，那么这个依赖属性将会收到变更通知

在我的应用里面，有很复杂的资源逻辑，我需要调试是哪个模块在修改资源，可以使用本文下面提供的方法

在开始之前，请先安装 Behavior 库，请通过 NuGet 安装 Microsoft.Xaml.Behaviors.Wpf 库。如果是 SDK 风格的 csproj 可以添加如下代码安装

```xml
  <ItemGroup>
    <PackageReference Include="Microsoft.Xaml.Behaviors.Wpf" Version="1.1.31" />
  </ItemGroup>
```

接下来编写 ResourceChangeEventBehavior 类用来监听动态资源变更

```csharp
    public class ResourceChangeEventBehavior : Behavior<FrameworkElement>
    {
        public static readonly DependencyProperty ResourceProperty = DependencyProperty.Register(
            "Resource", typeof(object), typeof(ResourceChangeEventBehavior), new PropertyMetadata(default(object), ResourceChangedCallback));

        public event EventHandler<ResourceChangedEventArgs> ResourceChanged;

        public object Resource
        {
            get { return GetValue(ResourceProperty); }
            set { SetValue(ResourceProperty, value); }
        }

        private static void ResourceChangedCallback(DependencyObject dependencyObject, DependencyPropertyChangedEventArgs args)
        {
            if (dependencyObject is ResourceChangeEventBehavior resourceChangeNotifier)
            {
                resourceChangeNotifier.OnResourceChanged(new ResourceChangedEventArgs(args.OldValue, args.NewValue));
            }
        }

        private void OnResourceChanged(ResourceChangedEventArgs args)
        {
            ResourceChanged?.Invoke(this, args);
        }
    }

    public class ResourceChangedEventArgs : EventArgs
    {
        public ResourceChangedEventArgs(object oldValue, object newValue)
        {
            OldValue = oldValue;
            NewValue = newValue;
        }

        public object OldValue { get; }
        public object NewValue { get; }
    }
```

可以看到实际使用的逻辑就是在 Resource 这个依赖属性变更的时候，触发 ResourceChanged 事件

以上就是所有的框架代码，使用方法如下，如下面代码将要监听 	VariableFontSize 这个动态资源的变更

```xml
    <i:Interaction.Behaviors>
      <resourceChangeEvent:ResourceChangeEventBehavior
        Resource="{DynamicResource VariableFontSize}"
        ResourceChanged="OnResourceChanged_VariableFontSize" />
    </i:Interaction.Behaviors>
```

全部的 XAML 代码如下

```xml
<Window x:Class="JeyaijikeneeWhejoniwairbu.MainWindow"
        xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        xmlns:d="http://schemas.microsoft.com/expression/blend/2008"
        xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006"
        xmlns:resourceChangeEvent="clr-namespace:JeyaijikeneeWhejoniwairbu"
        xmlns:i="http://schemas.microsoft.com/xaml/behaviors"
        xmlns:system="clr-namespace:System;assembly=System.Runtime"
        mc:Ignorable="d"
        Title="MainWindow" Height="450" Width="800">
  <Window.Resources>
    <system:Double x:Key="VariableFontSize">5</system:Double>
  </Window.Resources>
  <Grid>
    <i:Interaction.Behaviors>
      <resourceChangeEvent:ResourceChangeEventBehavior
        Resource="{DynamicResource VariableFontSize}"
        ResourceChanged="OnResourceChanged_VariableFontSize" />
    </i:Interaction.Behaviors>
    <Button Margin="10,10,10,10" HorizontalAlignment="Left" VerticalAlignment="Top" Content="Change Resource" Click="Button_OnClick"/>
  </Grid>
</Window>
```

在点击按钮变更动态资源

```csharp
        private void OnResourceChanged_VariableFontSize(object? sender, ResourceChangedEventArgs e)
        {

        }

        private void Button_OnClick(object sender, RoutedEventArgs e)
        {
            Resources["VariableFontSize"] = 10d;
        }
```

本文代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/fe0c808b/JeyaijikeneeWhejoniwairbu ) 欢迎下载执行，可以看到在点击按钮的时候进入 OnResourceChanged_VariableFontSize 方法

使用这个方法可以快速调试是在哪个模块变更了动态资源，也可以通过本文的方法在某个动态资源变更的时候执行其他逻辑，如动画

其实不使用 Behavior 库，使用附加属性也能实现相同的效果，但是使用 Behavior 库可以绑定到其他逻辑

特别感谢 [jeromerg](https://github.com/jeromerg ) 大佬提供的方法，详细请看 [https://github.com/jeromerg/WpfResourceChangeEvent](https://github.com/jeromerg/WpfResourceChangeEvent) 本文大部分代码都从这个仓库抄

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。

