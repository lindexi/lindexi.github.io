
# WPF 调试依赖属性变更方法

本文告诉大家如何调试 WPF 的某个依赖属性被变更的方法

<!--more-->


<!-- 发布 -->
<!-- 博客 -->
<!-- 标签：WPF，调试 -->

在 WPF 里面，所有的依赖属性都有带通知的功能，通过带通知的功能，可以在通知里加上断点，通过调用堆栈了解是哪个模块调用的

对依赖属性添加通知回调，可以使用如下方式

```csharp
            DependencyPropertyDescriptor.FromProperty(要监听的依赖属性, typeof(对象的类型)).AddValueChanged(要监听的对象,
                (sender, args) =>
                {
                    // 在这里加上断点
                });
```

例如调试 DataContext 的变更，可以使用如下代码

```csharp
            DependencyPropertyDescriptor.FromProperty(FrameworkElement.DataContextProperty, typeof(FrameworkElement)).AddValueChanged(要监听的对象,
                (sender, args) =>
                {
                    // 在这里加上断点
                });
```

添加断点之后，属性的变更有两个可能，第一个就是进入断点，第二个就是不进入断点，分别对应两个方向的调试。进入断点，可以通过 VisualStudio 的 调用堆栈 找到变更此依赖属性的方法，从而定位到是哪个模块变更了依赖属性

如果是不进入断点，可能是对象被换掉，试试在对应的属性上的 set 方法加上断点，或者在局部变量里面添加 Id 值，调试此变量的属性在哪被变更


一个用来调试的版本的代码放在[github](https://github.com/lindexi/lindexi_gd/tree/e284ff96734a84e9d4f49d76d5de06aa21e3423b/LalyiheahoLujarwallu) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/e284ff96734a84e9d4f49d76d5de06aa21e3423b/LalyiheahoLujarwallu) 欢迎访问

可以通过如下方式获取本文的源代码，先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin e284ff96734a84e9d4f49d76d5de06aa21e3423b
```

以上使用的是 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin e284ff96734a84e9d4f49d76d5de06aa21e3423b
```

获取代码之后，进入 LalyiheahoLujarwallu 文件夹




<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。