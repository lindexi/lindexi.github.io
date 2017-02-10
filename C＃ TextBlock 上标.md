# C＃ TextBlock 上标

我需要做一个函数，显示 $x^2$ ，但是看起来用 TextBlock 做的不好看。

我用 WPF 写的上标看起来不好看，但是最后有了一个简单方法让他好看。

本文告诉大家如何做一个好看的上标。

<!--more-->
<!-- csdn -->

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

![](http://7xqpl8.com1.z0.glb.clouddn.com/7abeb606-6faa-4f1e-ae7d-e19918db24e1QQ截图2017021015032520172101556.jpg)

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

代码一样，但是写的地方不一样，可以看到现在的上标就好看了。

这个问题暂时还没找到是为什么，但一定是垃圾WR弄的，于是我又可以去喷他了。如果有时间我去测下 UWP 的看是不是这样。

我把他传上 csdn ，大家可以下载来验证。

