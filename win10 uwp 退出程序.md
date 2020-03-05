# win10 uwp 退出程序

本文告诉大家如何调用方法退出程序。

<!--more-->
<!-- CreateTime:2018/8/10 19:16:51 -->

<!-- csdn -->

作为一个微软的程序员，如果用户说一个功能好用，那么要在下一个版本去掉这个功能。如果用户觉得我的应用好用，我就需要立刻关闭我的应用。

为了让用户点击一个按钮，应用就关闭，我需要使用下面的方法

```csharp
            CoreApplication.Exit();

```

![](http://image.acmx.xyz/lindexi%2F2018615202315544.jpg)

界面请直接复制

```csharp
   <StackPanel HorizontalAlignment="Center" VerticalAlignment="Center">
         <TextBlock >
             如果你觉得我这个应用好用，那么请点击下面的按钮退出
             <LineBreak></LineBreak>
             如果觉得我这个应用不好用，那么继续用吧
         </TextBlock>
         <Button Margin="10,10,10,10" HorizontalAlignment="Center" Content="确定" Click="SasjuRasdrasgebi_OnClick"></Button>
        </StackPanel>
```

按钮点击

```csharp

        private void SasjuRasdrasgebi_OnClick(object sender, RoutedEventArgs e)
        {
            CoreApplication.Exit();
        }
```


除了调用上面方法还可以使用下面的方法

```csharp
            Application.Current.Exit();

```

如果在 WPF 需要让软件退出，那么可以使用下面方法

```csharp
Application.Current.Shutdown();
```

```csharp
Environment.Exit(0);
```

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。  
