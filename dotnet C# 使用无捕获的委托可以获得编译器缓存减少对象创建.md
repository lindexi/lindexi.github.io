# dotnet C# 使用无捕获的委托可以获得编译器缓存减少对象创建

本文也叫跟着 Stephen Toub 大佬学性能优化系列，这是我从 Stephen Toub 大佬给 WPF 框架做性能优化学到的知识，通过让委托无捕获来进行性能优化

<!--more-->
<!-- CreateTime:2021/6/25 18:52:54 -->

<!-- 发布 -->

在 .NET 应用中，咱可以通过委托将某个方法传入到某个模块里面，让这个模块在合适的时候，执行传入的方法。如果传入的是实例的方法，那在编译器生成委托时，将会自动加上捕获包，本质上的捕获包是一个对象，也就是每次调用都需要分配对象

如以下代码

```csharp
        public void Foo(object _)
        {

        }

        public void AddFoo(Action<object> action, object state)
        {
            ActionList.Add((action, state));
        }
```

咱需要将 Foo 方法调用 AddFoo 方法，将 Foo 方法作为委托传入到 AddFoo 方法。此时咱有多个不同的方法来实现，为了让大家看到效果，咱来新建一个空白的 WPF 项目，在界面上放两个按钮

```xml
  <Grid>
    <StackPanel Orientation="Horizontal" HorizontalAlignment="Center" VerticalAlignment="Center">
      <Button Margin="10,10,10,10" Click="Button1_OnClick">点击</Button>
      <Button Margin="10,10,10,10" Click="Button2_OnClick">民与名扬</Button>
    </StackPanel>
  </Grid>
```

点击第一个按钮，此时咱使用简单的方法

```csharp
        private void Button1_OnClick(object sender, RoutedEventArgs e)
        {
            for (int i = 0; i < 100; i++)
            {
                AddFoo(Foo, null);
            }
        }
```

大部分的逻辑都是采用上面的方法传入的

但 [Stephen Toub](https://github.com/stephentoub ) 大佬在性能优化上，是使用了传入对象自身，减少委托捕获的方法来优化性能。在点击 民与名扬 按钮时，触发以下代码

```csharp
        private void Button2_OnClick(object sender, RoutedEventArgs e)
        {
            for (int i = 0; i < 100; i++)
            {
                AddFoo(s => ((MainWindow) s).Foo(null), this);
            }
        }
```

以上的逻辑有点绕，大概就是在 AddFoo 方法在设计上允许传入最后一个参数，最后的一个参数将会自动作为参数传入给委托，也就是代码的 s 变量的值就是 this 的值。此时的优化在于调用了 AddFoo 方法加入的委托不需要对 this 有任何的引用，因此就可以让 编译器 进行缓存，不需要每次都创建新的委托对象

咱来运行代码对比一下性能，运行代码，在看到 WPF 应用打开时，点击内存的获取快照，然后点击第一个按钮，再点击内存的获取快照，可以看到下图

<!-- ![](image/dotnet C# 使用无捕获的委托可以获得编译器缓存减少对象创建/dotnet C# 使用无捕获的委托可以获得编译器缓存减少对象创建1.png) -->

![](https://i.loli.net/2021/06/25/HCmE16t7fNXUpOQ.jpg)

可以看到加了 100 多个对象

然后点击 民与名扬 按钮，点击内存的获取快照，可以看到下图

<!-- ![](image/dotnet C# 使用无捕获的委托可以获得编译器缓存减少对象创建/dotnet C# 使用无捕获的委托可以获得编译器缓存减少对象创建2.png) -->

![](https://i.loli.net/2021/06/25/9DMhcZNzutoOI2F.jpg)

可以看到内存几乎没有添加任何对象。再多点击 民与名扬 按钮几次，点击内存的获取快照，可以看到几乎没有对象的分配。但是如果点击第一个按钮，点击内存的获取快照，可以看到内存加了很多对象

在性能优化时，可以考虑减少委托的捕获，如在传入实例的方法，也就是非静态的方法时，将会让委托捕获了 this 变量，需要创建委托。而如果 this 的变量是通过参数重新传入给委托的，此时可以做到不需要创建新的委托

这就是为什么有一些方法设计了委托传入，同时支持再传入一个 object 对象的原因。如 WPF 的 BeginInvoke 方法就是这样设计的，虽然咱现在推荐更多使用的是 InvokeAsync 方法

本文所有代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/fd1c53fefa4a6a27afeb785ee69f581f0af6ec5c/NewhearfarairchaylucoLerhejuche) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/fd1c53fefa4a6a27afeb785ee69f581f0af6ec5c/NewhearfarairchaylucoLerhejuche) 欢迎访问

可以通过如下方式获取本文的源代码，先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin fd1c53fefa4a6a27afeb785ee69f581f0af6ec5c
```

以上使用的是 gitee 的源，如果 gitee 不能访问，请替换为 github 的源

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
```

获取代码之后，进入 NewhearfarairchaylucoLerhejuche 文件夹

<!-- ![](image/dotnet C# 使用无捕获的委托可以获得编译器缓存减少对象创建/dotnet C# 使用无捕获的委托可以获得编译器缓存减少对象创建0.png) -->

![](https://i.loli.net/2021/06/25/EDoQvbcgafkI2SB.jpg)

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
