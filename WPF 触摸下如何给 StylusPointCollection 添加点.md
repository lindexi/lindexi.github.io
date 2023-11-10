# WPF 触摸下如何给 StylusPointCollection 添加点

本文告诉大家如何在触摸下给 WPF 的 StylusPointCollection 添加新的点

<!--more-->
<!-- 发布 -->
<!-- 博客 -->

在自己默认创建的 StylusPointCollection 里面添加点是十分简单的，如以下代码，可以非常简单添加到集合

```csharp
        StylusPointCollection stylusPointCollection = new StylusPointCollection();

        stylusPointCollection.Add(new StylusPoint(0, 0));
```

但是当你的 StylusPointCollection 是从 Stylus 事件里面获取的，比如以下代码的 StylusDown 事件里面获取的，那在添加点的时候可能你会收到 ArgumentException 异常

```csharp
    private void MainWindow_OnStylusDown(object sender, StylusDownEventArgs e)
    {
        StylusPointCollection stylusPointCollection = e.GetStylusPoints(this);

        stylusPointCollection.Add(new StylusPoint(0, 0));
    }
```

在一些触摸框下的设备，以上添加点的代码可能收到以下的异常信息

System.ArgumentException:“StylusPointDescriptions 不兼容。使用 StylusPointDescription.GetCommonDescription 方法查找公用 StylusPointDescription，然后调用 StylusPointCollection.Reformat，以返回兼容的 StylusPointCollection。 Arg_ParamName_Name”

这是因为从触摸拿到的 StylusPointCollection 预设了 StylusPointDescription 描述信息，而新创建的 StylusPoint 没有带上描述信息里面的内容，从而导致异常

在触摸下，收到的触摸点可以包含更多的信息，比如宽度高度、按钮点击状态等等。这些信息是要求整个 StylusPointCollection 里面的所有 StylusPoint 点都包含存在所声明的信息。想想，如果一个 StylusPointCollection 的描述里面说明点包含宽度信息，而如果其中某些点不包含，这要让 WPF 层如何能工作

这也就是为什么在一些触摸框下的设备才会抛出异常，一些触摸框下的设备不会抛出异常的原因。因为只有一些触摸框才会给触摸点带上更多的额外数据，如宽度高度等信息，在这些触摸框下的设备将由于创建的 StylusPoint 拿不到的额外描述信息，从而失败

可选的添加点到 StylusPointCollection 的方法有两个，一个是设置让 StylusPointCollection 去掉描述信息，另一个就是取现有的 StylusPoint 点复制其信息

先看第一个方法的实现

去掉 StylusPointCollection 的描述信息，可以通过 Reformat 方法设置一个空的 StylusPointDescription 去掉描述信息，如以下代码

```csharp
    private void MainWindow_OnStylusDown(object sender, StylusDownEventArgs e)
    {
        StylusPointCollection stylusPointCollection = e.GetStylusPoints(this);
        stylusPointCollection = stylusPointCollection.Reformat(new StylusPointDescription());

        stylusPointCollection.Add(new StylusPoint(0, 0));
    }
```

本文以上代码放在[github](https://github.com/lindexi/lindexi_gd/tree/5bbbe5643ee3fd484c1f36c79742399ba486fde6/RokelnejallwhuNeaferkairce) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/5bbbe5643ee3fd484c1f36c79742399ba486fde6/RokelnejallwhuNeaferkairce) 欢迎访问

可以通过如下方式获取本文的源代码，先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin 5bbbe5643ee3fd484c1f36c79742399ba486fde6
```

以上使用的是 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin 5bbbe5643ee3fd484c1f36c79742399ba486fde6
```

获取代码之后，进入 RokelnejallwhuNeaferkairce 文件夹

第二个方法是通过在 StylusPointCollection 里面存在的点的信息创建新的点。第二个方法比较黑科技，只适合用来不便修改原有的 StylusPointCollection 对象的情况，代码例子如下

```csharp
    private void MainWindow_OnStylusDown(object sender, StylusDownEventArgs e)
    {
        StylusPointCollection stylusPointCollection = e.GetStylusPoints(this);

        var stylusPoint = stylusPointCollection[0];
        stylusPoint.X = 1;
        stylusPoint.Y = 2;

        stylusPointCollection.Add(stylusPoint);
    }
```

由于 StylusPoint 是一个结构体，根据 C# 基础知识，结构体获取的时候都是一次浅拷贝，也就是通过 `stylusPointCollection[0]` 所获取返回的 StylusPoint 已经和原本在集合里面的点是两个不同的点了，对 `stylusPoint` 局部变量的任何更改都不会影响到原本的点

因此通过此方式即可方便的进行 StylusPoint 的拷贝，通过拷贝的方式获取到必要的额外描述信息。使用结构体特性进行拷贝而不是重新创建的方法，即可让点包含触摸点集合所描述的信息，从而可以正确加入到集合里面

但无论如何，新添加的点的信息肯定是模拟出来的，这就意味着对于模拟出来的点的额外信息在你的具体业务上的处理，是必须要符合你的预期的。假定你本身就拿着触摸的面积进行一些业务处理，那如果新加的点使用了诡异的数据，那自然将会让你这部分业务不能符合预期

当然了，自己创建点的时候，添加上足够的描述信息也是可以的，只不过这部分代码不好写，且实现效果和以上第二个方法差不多