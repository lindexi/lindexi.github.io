---
title: Avalonia 已知问题 第二次 Composition Animation 无法播放
description: 在 Avalonia 里面，如果多次加入 Composition 的 Animation 动画，将会看到第二次加入是停止播放动画的
tags: Avalonia
category: 
---

<!-- CreateTime:2024/10/29 07:01:04 -->

<!-- 发布 -->
<!-- 博客 -->

这是 Avalonia 的已知问题，我已经报告给官方，详细请看 <https://github.com/AvaloniaUI/Avalonia/pull/17370>

我尝试修复了此问题，请看 <https://github.com/AvaloniaUI/Avalonia/pull/17370>

此问题我在 11.0-11.1 版本复现，其他版本我没有进行测试

复现的步骤如下：

1. 在界面放入一个 UI 控件，如 Border 控件
2. 通过 `ElementComposition.GetElementVisual` 方法获取 CompositionVisual 对象，再使用此对象创建和播放一个 Vector3DKeyFrameAnimation 动画
3. 重复执行步骤 2

此时你可以看到重复执行步骤 2 时，原本正在播放的动画已经停止播放了

以下是我的 XAML 界面代码

```xml
    <Grid>
        <Border x:Name="ScanBorder" ZIndex="101" IsVisible="True" HorizontalAlignment="Center" VerticalAlignment="Top"
                Height="220" Width="600">
            <Border.Background>
                <LinearGradientBrush StartPoint="0%,0%" EndPoint="0%,100%">
                    <GradientStop Color="#0033CEFF" Offset="0" />
                    <GradientStop Color="#CC3592FF" Offset="1" />
                </LinearGradientBrush>
            </Border.Background>
            <Border.RenderTransform>
                <TranslateTransform />
            </Border.RenderTransform>
        </Border>

        <Button x:Name="ControlButton" Content="Click" Click="ControlButton_OnClick"></Button>
    </Grid>
```

以下是我的 C# 代码

```csharp
    private Vector3DKeyFrameAnimation? _vector3DKeyFrameAnimation;
    private CompositionVisual? _scanBorderCompositionVisual;

    private void ControlButton_OnClick(object? sender, RoutedEventArgs e)
    {
        _scanBorderCompositionVisual = ElementComposition.GetElementVisual(ScanBorder)!;
        var compositor = _scanBorderCompositionVisual.Compositor;
        _vector3DKeyFrameAnimation = compositor.CreateVector3DKeyFrameAnimation();
        _vector3DKeyFrameAnimation.InsertKeyFrame(0f, _scanBorderCompositionVisual.Offset with { Y = 0 });
        _vector3DKeyFrameAnimation.InsertKeyFrame(1f, _scanBorderCompositionVisual.Offset with { Y = this.Bounds.Height - ScanBorder.Height });
        _vector3DKeyFrameAnimation.Duration = TimeSpan.FromSeconds(2);
        _vector3DKeyFrameAnimation.IterationBehavior = AnimationIterationBehavior.Count;
        _vector3DKeyFrameAnimation.IterationCount = 30;

        _scanBorderCompositionVisual.StartAnimation("Offset", _vector3DKeyFrameAnimation);
    }
```

我将最简复现步骤的例子项目上传到 [github](https://github.com/lindexi/lindexi_gd/tree/f82af28bab6f5cdfbd13c48c19b4f0a21a50ae06/AvaloniaIDemo/JallkeleejurCihayaiqalker) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/f82af28bab6f5cdfbd13c48c19b4f0a21a50ae06/AvaloniaIDemo/JallkeleejurCihayaiqalker) 上，可以使用如下命令行拉取代码。我整个代码仓库比较庞大，使用以下命令行可以进行部分拉取，拉取速度比较快

先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin f82af28bab6f5cdfbd13c48c19b4f0a21a50ae06
```

以上使用的是国内的 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码，将 gitee 源换成 github 源进行拉取代码。如果依然拉取不到代码，可以发邮件向我要代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin f82af28bab6f5cdfbd13c48c19b4f0a21a50ae06
```

获取代码之后，进入 AvaloniaIDemo/JallkeleejurCihayaiqalker 文件夹，即可获取到源代码

预期的行为是能够控制 Composition 的 Animation 动画的停止以及开启新的动画

根据我的分析问题原因是在更基础的 InlineDictionary 在处理单项重新赋值时的不正确行为，让动画模块第二次进入时不能符合预期工作

根据阅读 Avalonia 的代码可以看到 InlineDictionary 在只有单项的行为是通过 Set 方法调用进入时，将会忽略 overwrite 参数，从而导致 InlineDictionary 只有一项时，再次调用 Set 时的效果将会和调用 Add 方法相同。此行为将导致 composition animation 动画播放行为不符合预期，将导致第二次的 composition animation 无法播放。为什么第二次的 composition animation 无法播放？原因是第二次准备播放的 composition animation 无法将第一次的 composition animation 替换掉，而是将第二次的 composition animation 加入到第一次的 composition animation 后面，从而导致第二次设置的 composition animation 无法被执行

核心代码如下

```csharp
internal struct InlineDictionary<TKey, TValue> : IEnumerable<KeyValuePair<TKey, TValue>> where TKey : class
{
    public void Set(TKey key, TValue value) => SetCore(key, value, true);

    public TValue this[TKey key]
    {
        get
        {
            if (TryGetValue(key, out var rv))
                return rv;
            throw new KeyNotFoundException();
        }
        set => Set(key, value);
    }

    void SetCore(TKey key, TValue value, bool overwrite)
    {
        if (key == null)
            throw new ArgumentNullException();
        if (_data == null)
        {
            _data = key;
            _value = value;
        } 
        else if (_data is KeyValuePair[] arr)
        {
            ...
        }
        else if (_data is Dictionary<TKey, TValue?> dic)
        {
            ...
        }
        else
        {
            // We have a single element, upgrade to array.
            arr = new KeyValuePair[6];
            arr[0] = new KeyValuePair((TKey)_data, _value);
            arr[1] = new KeyValuePair(key, value);
            _data = arr;
            _value = default;
        }
    }
}
```

通过以上代码分析可以看到，在 InlineDictionary 首次加入时，将会进入 `if (_data == null)` 分支，使用如下代码分别给 `_data` 和 `_value` 赋值

但是第二次进来的时候，将会进入 `else` 分支，在这个分支里面啥都判断，没有判断 `overwrite` 和 `key` 的值，直接就创建为 KeyValuePair 数组。这就意味着第二次进入的时候，将让 Set 方法和 Add 方法相同，都是做添加而不是替换

这就导致了在 Composition 的 Animation 动画里面第二次设置动画的时候，停止播放动画

如以下的 [ServerObjectAnimations](https://github.com/AvaloniaUI/Avalonia/blob/master/src/Avalonia.Base/Rendering/Composition/Server/ServerObjectAnimations.cs#L109-L120) 代码，可以看到在加入动画的时候，先获取旧的代码，将其调用 Deactivate 停下，再将其赋值为新的动画

```csharp
class ServerObjectAnimations
{
    ... // 忽略其他代码
    private InlineDictionary<CompositionProperty, ServerObjectAnimationInstance> _animations;

    public void OnSetAnimatedValue<T>(CompositionProperty<T> prop, ref T field, TimeSpan committedAt, IAnimationInstance animation) where T : struct
    {
        if (_owner.IsActive && _animations.TryGetValue(prop, out var oldAnimation))
            oldAnimation.Animation.Deactivate();
        _animations[prop] = new ServerObjectAnimationInstance<T>(this, animation, prop);
            
        animation.Initialize(committedAt, ExpressionVariant.Create(field), prop);
        if(_owner.IsActive)
            animation.Activate();
            
        OnSetDirectValue(prop);
    }
}
```

由于 InlineDictionary 存在问题，只有一项的时候，赋值进入第二项时，做的是添加第二项但不删除第一项。这就导致第二次加入动画时候，第一个动画被停止，但是第一个动画还在字典里面，后续获取将会返回第一个动画。第二个动画将不会被返回。这就是为什么第二次的动画无法播放的原因
