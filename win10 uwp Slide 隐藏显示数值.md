# win10 uwp slider 隐藏显示数值

本文告诉大家，如何隐藏 slider 拖动出现的数值。

因为这个数值是控件给的，样子不好看，而且在 slider 的上面，不是在他的右边，所以需要隐藏他。

<!--more-->
<!-- CreateTime:2018/8/10 19:17:19 -->
 
<!-- csdn -->

![](http://image.acmx.xyz/34fdad35-5dfe-a75b-2b4b-8c5e313038e2%2F201811292241.jpg)

最简单的方法是设置 IsThumbToolTipEnabled ，请看下面

```csharp
<Slider IsThumbToolTipEnabled="False"/>
```

![](http://image.acmx.xyz/34fdad35-5dfe-a75b-2b4b-8c5e313038e2%2F201811292330.jpg)

如果这时需要显示拖动的值，可以使用 ValueChanged 事件，我下面使用一个 TextBlock 来显示拖动的值

```csharp
     <Slider Margin="10,10,10,10" IsThumbToolTipEnabled="False" ValueChanged="Slider_OnValueChanged"></Slider>

        <TextBlock x:Name="CumjvpxVufe" Margin="10,100,10,10"></TextBlock>
```

![](http://image.acmx.xyz/34fdad35-5dfe-a75b-2b4b-8c5e313038e2%2F20181129288.jpg)

如果需要绑定，那么就可以不需要后台写代码

```csharp
   <Slider x:Name="CumjvpxVufe" Margin="10,10,10,10" IsThumbToolTipEnabled="False" ></Slider>

        <TextBlock Margin="10,100,10,10" Text="{x:Bind CumjvpxVufe.Value,Mode=OneWay}"></TextBlock>
```

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。