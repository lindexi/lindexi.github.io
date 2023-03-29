# win10 uwp 笔迹书写预测 墨迹书写加速

在 UWP 的 InkCanvas 里自带了预测书写轨迹的功能，开启此功能可以进行书写预测，从而减少书写延迟。本文将告诉大家如何在 UWP 的 InkCanvas 里开启笔迹书写预测功能

<!--more-->
<!-- CreateTime:2023/3/28 14:06:26 -->

<!-- 发布 -->
<!-- 博客 -->

在 UWP 的 InkCanvas 里开启笔迹书写预测功能只需要设置 InkModelerAttributes 的 PredictionTime 属性即可，这个属性是用来设置预测的时间量。默认值为 15 毫秒，有效范围为 0 到 20 毫秒，也就是最多能预测 20 毫秒之后的轨迹点。印象中在 10240 的文档里面，是有一个使用 Inertia 惯性预测算法的笔迹书写预测的，但是我现在还没找到文档

设置 InkModelerAttributes 的 PredictionTime 属性需要先获取到 InkDrawingAttributes 对象，因为 InkModelerAttributes 是 InkDrawingAttributes 里的一个属性

演示的项目如下，先在 MainPage.xaml 添加以下代码

```xml
    <Grid>
        <InkCanvas x:Name="InkCanvas"></InkCanvas>
    </Grid>
```

以上代码在界面里存放一个 InkCanvas 元素

接着在构造函数使用以下代码设置笔迹书写预测

```csharp
            var inkPresenter = InkCanvas.InkPresenter;
            inkPresenter.InputDeviceTypes =
                CoreInputDeviceTypes.Touch | CoreInputDeviceTypes.Mouse | CoreInputDeviceTypes.Pen;
            var defaultDrawingAttributes = inkPresenter.CopyDefaultDrawingAttributes();
            defaultDrawingAttributes.Color = Colors.Red;
            defaultDrawingAttributes.ModelerAttributes.PredictionTime = TimeSpan.FromMilliseconds(20);
            inkPresenter.UpdateDefaultDrawingAttributes(defaultDrawingAttributes);
```

以上核心代码就是通过 `defaultDrawingAttributes.ModelerAttributes.PredictionTime = TimeSpan.FromMilliseconds(20);` 设置预测时间量。设置为最大的 20 毫秒

如此即可完成笔迹书写预测功能，运行代码，在屏幕上用触摸书写，可以看到在实际鼠标光标之前就能画出一段在不断变更的笔迹

修改 PredictionTime 属性的值，可以在性能比较差的设备上看到比较明显的效果

这个技术在许多 UWP 应用上，例如 OneNote UWP 等都是默认开启的

但是毕竟是预测的，肯定存在纠错时候，这时将会发现笔迹的笔尖抖动。也就是开启笔迹预测功能，能够降低延迟，但是可能在预测和实际触摸点不符合时笔尖会变更

详细请参阅 [InkModelerAttributes.PredictionTime - Windows UWP applications Microsoft Learn](https://learn.microsoft.com/zh-cn/uwp/api/windows.ui.input.inking.inkmodelerattributes.predictiontime)

如期望更换预测算法或者做更底层控制，请参阅 [DelegatedInkTrailVisual.AddTrailPointsWithPrediction Method (Windows.UI.Composition) - Windows UWP applications Microsoft Learn](https://learn.microsoft.com/en-us/uwp/api/windows.ui.composition.delegatedinktrailvisual.addtrailpointswithprediction )

本文代码放在[github](https://github.com/lindexi/lindexi_gd/tree/b8cb3470fa7831173d617aad6498b067236edad2/KalljelcufedilaiCaiheebeferwhearlair) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/b8cb3470fa7831173d617aad6498b067236edad2/KalljelcufedilaiCaiheebeferwhearlair) 欢迎访问

可以通过如下方式获取本文以上的源代码，先创建一个名为 KalljelcufedilaiCaiheebeferwhearlair 的空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin b8cb3470fa7831173d617aad6498b067236edad2
```

以上使用的是 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin b8cb3470fa7831173d617aad6498b067236edad2
```

获取代码之后，进入 KalljelcufedilaiCaiheebeferwhearlair 文件夹

更多触摸和笔迹书写相关请看 [WPF 触摸相关](https://blog.lindexi.com/post/WPF-%E8%A7%A6%E6%91%B8%E7%9B%B8%E5%85%B3.html)
