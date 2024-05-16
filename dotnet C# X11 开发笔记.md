# dotnet C# X11 开发笔记

本文记录我学习开发 X11 应用的笔记

<!--more-->
<!-- CreateTime:2024/04/02 07:07:16 -->

<!-- 发布 -->
<!-- 博客 -->

## 如何设置X11里面两个窗口之间的层级关系

如何类似 WPF 的 Owner 之类的关系？可使用 XSetTransientForHint 方法。比如有 a 和 b 两个窗口，使用下面代码即可设置 a 窗口一定在 b 窗口上方

```csharp
        // 我们使用XSetTransientForHint函数将窗口a设置为窗口b的子窗口。这将确保窗口a始终在窗口b的上方
        XSetTransientForHint(Display, a, b);
```

以上代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/0331c5dd6057106df5cb179e45d34966a3eafd1b/GececurbaiduhaldiFokeejukolu) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/0331c5dd6057106df5cb179e45d34966a3eafd1b/GececurbaiduhaldiFokeejukolu) 上，可以使用如下命令行拉取代码

先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin 0331c5dd6057106df5cb179e45d34966a3eafd1b
```

以上使用的是 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码，将 gitee 源换成 github 源进行拉取代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin 0331c5dd6057106df5cb179e45d34966a3eafd1b
```

获取代码之后，进入 GececurbaiduhaldiFokeejukolu 文件夹，即可获取到源代码

## 设置窗口无边框

设置窗口的override_redirect属性为True，以避免窗口管理器的干预，如此即可实现无边框

```csharp
        var valueMask =
            0
            | SetWindowValuemask.OverrideRedirect;
        var xSetWindowAttributes = new XSetWindowAttributes
        {
            override_redirect = true, // 设置窗口的override_redirect属性为True，以避免窗口管理器的干预
        };

        var handle = XCreateWindow(Display, rootWindow, 100, 100, 1000, 500, 5,
            32,
            (int) CreateWindowArgs.InputOutput,
            visual,
            (nuint) valueMask, ref xSetWindowAttributes);
```

以上代码的 SetWindowValuemask.OverrideRedirect 十分重要，如果没有加上将会导致 `override_redirect` 设置无效

以上代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/dc1b79521e00300dfaef49d54226b6f687b25b3e/GececurbaiduhaldiFokeejukolu) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/dc1b79521e00300dfaef49d54226b6f687b25b3e/GececurbaiduhaldiFokeejukolu) 上，可以使用如下命令行拉取代码

先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin dc1b79521e00300dfaef49d54226b6f687b25b3e
```

以上使用的是 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码，将 gitee 源换成 github 源进行拉取代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin dc1b79521e00300dfaef49d54226b6f687b25b3e
```

获取代码之后，进入 GececurbaiduhaldiFokeejukolu 文件夹，即可获取到源代码

## 移动窗口

核心 C# 代码如下，即可移动 X11 窗口

```csharp
XMoveWindow(display, handle, Random.Shared.Next(500), Random.Shared.Next(200));
```

完全的代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/8b5f024b002b2dbc2bd630c2ffcd24ece9b5a9c5/DikalehebeekaJaqunicobo) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/8b5f024b002b2dbc2bd630c2ffcd24ece9b5a9c5/DikalehebeekaJaqunicobo) 上，可以使用如下命令行拉取代码

先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin 8b5f024b002b2dbc2bd630c2ffcd24ece9b5a9c5
```

以上使用的是 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码，将 gitee 源换成 github 源进行拉取代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin 8b5f024b002b2dbc2bd630c2ffcd24ece9b5a9c5
```

获取代码之后，进入 DikalehebeekaJaqunicobo 文件夹，即可获取到源代码

设置窗口坐标过程中，是不会存在窗口移动动画的

## 设置窗口透明

参阅： [dotnet C# 设置 X11 应用窗口背景透明](https://blog.lindexi.com/post/dotnet-C-%E8%AE%BE%E7%BD%AE-X11-%E5%BA%94%E7%94%A8%E7%AA%97%E5%8F%A3%E8%83%8C%E6%99%AF%E9%80%8F%E6%98%8E.html )


## 和 Avalonia 相互调用

设置工具栏与 X11 窗口绘制的笔迹关联，要求 X11 笔迹窗口在下方，配合设置X11里面两个窗口之间的层级关系的方法即可实现

获取 Avalonia 的 X11 窗口，代码如下

```csharp
    if (TryGetPlatformHandle()?.Handle is { } handle)
    {
    }
```

以上代码拿到的 handle 就是可以用来作为 X11 窗口的指针，类似于 Windows 下的窗口句柄概念

以上代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/e42ddbb8989ca0dd7e859dd6fd9cb0ddbb4d3fd1/GececurbaiduhaldiFokeejukolu) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/e42ddbb8989ca0dd7e859dd6fd9cb0ddbb4d3fd1/GececurbaiduhaldiFokeejukolu) 上

## 和 UNO Gtk 相互调用

参阅 [dotnet 如何从 Gtk 3 的窗口到对应的 X11 窗口](https://blog.lindexi.com/post/dotnet-%E5%A6%82%E4%BD%95%E4%BB%8E-Gtk-3-%E7%9A%84%E7%AA%97%E5%8F%A3%E5%88%B0%E5%AF%B9%E5%BA%94%E7%9A%84-X11-%E7%AA%97%E5%8F%A3.html )


## 更多博客

[dotnet X11 窗口之间发送鼠标消息 模拟鼠标输入](https://blog.lindexi.com/post/dotnet-X11-%E7%AA%97%E5%8F%A3%E4%B9%8B%E9%97%B4%E5%8F%91%E9%80%81%E9%BC%A0%E6%A0%87%E6%B6%88%E6%81%AF-%E6%A8%A1%E6%8B%9F%E9%BC%A0%E6%A0%87%E8%BE%93%E5%85%A5.html )

[dotnet 后台线程发送 X11 窗口消息](https://blog.lindexi.com/post/dotnet-%E5%90%8E%E5%8F%B0%E7%BA%BF%E7%A8%8B%E5%8F%91%E9%80%81-X11-%E7%AA%97%E5%8F%A3%E6%B6%88%E6%81%AF.html )

[dotnet 后台线程设置 X11 窗口最小化](https://blog.lindexi.com/post/dotnet-%E5%90%8E%E5%8F%B0%E7%BA%BF%E7%A8%8B%E8%AE%BE%E7%BD%AE-X11-%E7%AA%97%E5%8F%A3%E6%9C%80%E5%B0%8F%E5%8C%96.html )
