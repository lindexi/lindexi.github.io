# win10 uwp 好看的时间选择控件

本文告诉大家我找到的好看的时间选择控件。

<!--more-->
<!-- CreateTime:2019/8/30 8:57:20 -->


<div id="toc"></div>

先给大家看一下图，然后就知道我说的是什么

![](https://i.loli.net/2018/06/15/5b23b83d42083.gif)

首先需要安装 Nuget ，搜索 `DeanChalk.UWP.TimePicker` 或输入`Install-Package DeanChalk.UWP.TimePicker -Version 1.0.0` 安装必要的库

几乎所有在 xaml 引用其他大佬的库都需要先添加引用，这里需要添加的引用请看下面代码

```csharp
    xmlns:deanChalk="using:DeanChalk.UWP.TimePicker"

```

在加上引用之后要使用这个新的控件是非常简单，只需要下面一句代码

```csharp
         <deanChalk:TimePicker x:Name="TimePicker" BorderThickness="0"></deanChalk:TimePicker>

```

这时可以把 TimePicker 的 Time 和原生的 TimePicker 绑定，当然现在绑定了时间控件还不能运行

```csharp
         <TimePicker Margin="10,10,10,10" HorizontalAlignment="Center" Time="{x:Bind TimePicker.Time,Mode=TwoWay}"></TimePicker>
```

如果要运行代码，那么请复制我的代码

```csharp
     <StackPanel HorizontalAlignment="Center" VerticalAlignment="Center">
         <deanChalk:TimePicker x:Name="TimePicker" BorderThickness="0"></deanChalk:TimePicker>
         <TimePicker Margin="10,10,10,10" HorizontalAlignment="Center" Time="{x:Bind TimePicker.Time,Mode=TwoWay}"></TimePicker>
         <TextBlock >
             如果你觉得我这个应用好用，那么请点击下面的按钮退出
             <LineBreak></LineBreak>
             如果觉得我这个应用不好用，那么继续用吧
         </TextBlock>
         <Button Margin="10,10,10,10" HorizontalAlignment="Center" Content="确定" Click="SasjuRasdrasgebi_OnClick"></Button>
        </StackPanel>

```

上面的代码是不需要后台代码，大家只要把代码复制在 MainPage.xaml 文件运行就可以看到上面的图片

这里点击按钮退出软件请看 [How to use code to exit the application in UWP](https://lindexi.oschina.io/lindexi/post/How-to-use-code-to-exit-the-application-in-UWP.html )

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。  

