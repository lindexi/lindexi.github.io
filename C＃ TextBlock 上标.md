# C＃ TextBlock 上标

我需要做一个函数，显示 $x^2$ ，但是看起来用 TextBlock 做的不好看。

我用 WPF 写的上标看起来不好看，但是最后有了一个简单方法让他好看。

本文告诉大家如何做一个好看的上标。

<!--more-->
<!-- CreateTime:2018/8/10 19:16:52 -->


<div id="toc"></div>

一开始做的方法：

把下面代码写在页面里，使用对齐是上面，改变字号，于是看起来就是上标。

```xml
          <TextBlock x:Name="TextBlock">
            <Run Text="y=x"></Run>
            <Run Text="2" BaselineAlignment="TextTop"
                 FontSize="8"></Run>
        </TextBlock>
```

于是看起来：

![](http://image.acmx.xyz/7abeb606-6faa-4f1e-ae7d-e19918db24e1QQ截图2017021015032520172101556.jpg)

其实已经可以了，但是发现距离很大，那么如何让距离变小？

我找了很久，发现可以在 xaml.cs 上写。


```csharp
            var textBlock = TextBlock;
            textBlock.Inlines.Add(new Run("y = "));
            textBlock.Inlines.Add(new Run("x"));
            Run run=new Run();
            run.FontSize = 7;
            run.BaselineAlignment = BaselineAlignment.TextTop;
            run.Text = "2";
            textBlock.Inlines.Add(run);
```

![](http://image.acmx.xyz/7abeb606-6faa-4f1e-ae7d-e19918db24e1QQ截图201702101503252017210152621.jpg)

代码一样，但是写的地方不一样，可以看到现在的上标就好看了。


UWP 上标也一样。为什么写在 Xaml 间隔会那么大，是不是WR弄的？其实试试下面代码，注意不要格式化，直接写的样子和我的一样试试。


```csharp
          <TextBlock x:Name="TextBlock">
            <TextBlock.Inlines>
                <Run Text="y=x"/><Run Text="2" BaselineAlignment="TextTop"
                 FontSize="8"/>
            </TextBlock.Inlines>
        </TextBlock>
```

![](http://image.acmx.xyz/7abeb606-6faa-4f1e-ae7d-e19918db24e1QQ截图201702101503252017210153833.jpg)

原因就是Run写在两行，会把换行给记下，于是间隔就大了，写在一起的Run就不会出现这个距离。

但是我的 格式化会把Run放在下一行，所以可能我这里看的好的，在你这就会换行，看起来上标就有了距离。

我把他传上 csdn ，大家可以下载来验证。

代码：[http://download.csdn.net/detail/lindexi_gd/9751879](http://download.csdn.net/detail/lindexi_gd/9751879)

<script type="text/javascript"
 src="http://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-AMS-MML_HTMLorMML">
</script>

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。  