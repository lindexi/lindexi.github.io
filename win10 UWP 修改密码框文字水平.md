# win10 UWP 修改密码框文字水平

一般的密码框输入左对齐，如何让他不左对齐

![](http://image.acmx.xyz/AwCCAwMAItoFAMV%2BBQA28wYAAQAEAK4%2BAQBmQwIAaOgJAOjZ%2F2017315175218.jpg)

![](http://image.acmx.xyz/AwCCAwMAItoFAMV%2BBQA28wYAAQAEAK4%2BAQBmQwIAaOgJAOjZ%2F2017315175617.jpg)

<!--more-->
<!-- CreateTime:2018/8/10 19:17:19 -->


<div id="toc"></div>
<!-- csdn -->

设置了`HorizontalContentAlignment`没有变化

于是找了很久，有一个方法可以设置，下面将会说这个方法

新建一个 PasswordBox 控件，编辑模板，可以看到 vs 自动写出很多代码

可以看到这样代码，需要自己去找一下


```csharp
    
    
    <ScrollViewer x:Name="ContentElement" AutomationProperties.AccessibilityView="Raw" HorizontalScrollMode="{TemplateBinding ScrollViewer.HorizontalScrollMode}" HorizontalScrollBarVisibility="{TemplateBinding ScrollViewer.HorizontalScrollBarVisibility}" IsTabStop="False" IsHorizontalRailEnabled="{TemplateBinding ScrollViewer.IsHorizontalRailEnabled}" IsVerticalRailEnabled="{TemplateBinding ScrollViewer.IsVerticalRailEnabled}" Margin="{TemplateBinding BorderThickness}" Padding="{TemplateBinding Padding}" Grid.Row="1" VerticalScrollBarVisibility="{TemplateBinding ScrollViewer.VerticalScrollBarVisibility}" VerticalScrollMode="{TemplateBinding ScrollViewer.VerticalScrollMode}" ZoomMode="Disabled"/>
```
但是需要修改的只有`HorizontalAlignment`，
修改这句就好，使用下面代码 代替上面的代码


```csharp
                                <ScrollViewer x:Name="ContentElement" VerticalAlignment="Center" HorizontalAlignment="Stretch" AutomationProperties.AccessibilityView="Raw" HorizontalScrollMode="{TemplateBinding ScrollViewer.HorizontalScrollMode}" HorizontalScrollBarVisibility="{TemplateBinding ScrollViewer.HorizontalScrollBarVisibility}" 
                                          IsTabStop="False" 
                                          IsHorizontalRailEnabled="{TemplateBinding ScrollViewer.IsHorizontalRailEnabled}" IsVerticalRailEnabled="{TemplateBinding ScrollViewer.IsVerticalRailEnabled}" Margin="{TemplateBinding BorderThickness}" Padding="{TemplateBinding Padding}" Grid.Row="1" VerticalScrollBarVisibility="{TemplateBinding ScrollViewer.VerticalScrollBarVisibility}" VerticalScrollMode="{TemplateBinding ScrollViewer.VerticalScrollMode}" ZoomMode="Disabled"/>

```

修改他的 `HorizontalAlignment` 修改为 Center ，密码控件就是上面的图，密码中间。

https://stackoverflow.com/questions/42805582/setting-the-alignment-of-a-passwordbox-in-uwp

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。 