---
title: WPF UNO 测试固定尺寸且水平和垂直对齐设置 Stretch 的元素在容器内的布局行为
description: 本文将告诉大家我对 WPF 的自定义布局容器和自定义控件进行的布局行为测试中的一个小点，即测试固定元素的尺寸的情况下或元素尺寸为有限尺寸的情况下，同步设置元素的水平和垂直对齐为 Stretch 来测试元素在容器内的布局行为，元素分别在容器给元素的布局尺寸大于元素的尺寸和小于元素尺寸的行为
tags: WPF,UNO
category: 
---

<!-- CreateTime:2024/1/26 19:25:10 -->

<!-- 发布 -->
<!-- 博客 -->

由于刚好运行在 WPF 之上 UNO 框架里的元素行为和 WPF 原生布局行为是完全相同的，本文也作为 UNO 的元素布局测试记录内容

如下面代码，编写一个自定义的继承于 Panel 类型的自定义布局容器，重写布局容器设置其布局行为为将自身的尺寸传入给到里层控件

```csharp
    protected override Size MeasureOverride(Size availableSize)
    {
        return availableSize;
    }

    protected override Size ArrangeOverride(Size finalSize)
    {
        Children[0].Arrange(new Rect(0, 0, finalSize.Width, finalSize.Height));

        return finalSize;
    }
```

这就意味着如果将此自定义容器放入到窗口里面，那就可以通过修改窗口的尺寸进而修改到此自定义容器的尺寸，从而测试在自定义容器给里层元素不同的布局空间时，设置了水平和垂直对齐为 Stretch 的元素会如何布局

给以上的这个自定义容器插入一个元素，设置元素给定尺寸且设置了水平和垂直对齐，如下面代码

```csharp
        var grid = new Grid()
        {
            Background = new SolidColorBrush(Colors.Black),
            Width = 500,
            Height = 500,
            HorizontalAlignment = HorizontalAlignment.Stretch,
            VerticalAlignment = VerticalAlignment.Stretch,
        }
```

为了更好的进行测试，我还给以上的 Grid 添加一圈的带背景的 Border 控件，用来测试在布局尺寸空间超过元素所需尺寸时的行为，和测试在布局尺寸空间小于元素所需尺寸时的压缩元素裁剪行为

对 WPF 和跑在 WPF 框架之上的 UNO 框架的测试行为都符合下图

<!-- ![](image/WPF UNO 测试固定尺寸且水平和垂直对齐设置 Stretch 的元素在容器内的布局行为/WPF UNO 测试固定尺寸且水平和垂直对齐设置 Stretch 的元素在容器内的布局行为0.gif) -->
![](https://img2023.cnblogs.com/blog/1080237/202409/1080237-20240918203538159-127544524.gif)

根据上图可以知道，当上层容器给定元素的可布局尺寸大于元素所需尺寸时，元素将会进行居中。当上层容器给定元素的可布局尺寸小于元素所需尺寸时，元素行为将和左上对齐时相同

本文以上代码放在[github](https://github.com/lindexi/lindexi_gd/tree/dc173cdd8cea18bdbec9c99f127252efd4f4a5f8/Laicairqechear) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/dc173cdd8cea18bdbec9c99f127252efd4f4a5f8/Laicairqechear) 欢迎访问

可以通过如下方式获取本文的源代码，先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin dc173cdd8cea18bdbec9c99f127252efd4f4a5f8
```

以上使用的是 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin dc173cdd8cea18bdbec9c99f127252efd4f4a5f8
```

获取代码之后，进入 Laicairqechear 文件夹
