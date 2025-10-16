---
title: C# 使用 using 关键字间接实现只读局部变量的方法
description: 众所周知，在 C# 里面是没有只读局部变量的功能的。但有趣的 C# 语法让咱可以使用现有的 using 关键字间接实现只读局部变量
tags: C#
category: 
---

<!-- 发布 -->
<!-- 博客 -->

在 C# 里面引入只读局部变量是一个存在 C# 语言设计仓库里面很久的讨论，详细请看 <https://github.com/dotnet/csharplang/discussions/8479>

官方在 2024.10.04 明确表示不会为 C# 添加只读局部变量的语法支持

今天我看到了一位名为 [Sator Imaging](https://github.com/sator-imaging) 的开发者给出了一个很好的实现方案，那就是使用 using 关键字间接实现只读局部变量。以下是我根据他的想法编写的代码

```csharp
void Foo()
{
    using var iro = 10.AsReadOnly();
    using var fro = 1.1f.AsReadOnly();

    int foo = iro;
    int bar = 20 * iro;  // can use as 'int'
    float baz = fro * fro * iro;

    // error: 'using variable' is protected by the system
    //iro = 20.AsReadOnly();
}

public readonly record struct ReadOnly<T>(T Value) : IDisposable
{
    void IDisposable.Dispose() { }
    public static implicit operator T(ReadOnly<T> x) => x.Value;
}

public static class ReadOnly
{
    public static ReadOnly<T> AsReadOnly<T>(this T value) => new(value);
}
```

这个方法巧妙在于利用了 C# 标记了 using 的变量不允许被更改的现有功能。通过结构体包装现有的局部变量类型对象，再配合隐式转换，从而实现比较弱的侵入性

本文代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/c73e4993fdc7fb8cdea2ab8367ba004003a94c27/Workbench/FacikereruyekoKiherbawjaiwha) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/c73e4993fdc7fb8cdea2ab8367ba004003a94c27/Workbench/FacikereruyekoKiherbawjaiwha) 上，可以使用如下命令行拉取代码。我整个代码仓库比较庞大，使用以下命令行可以进行部分拉取，拉取速度比较快

先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin c73e4993fdc7fb8cdea2ab8367ba004003a94c27
```

以上使用的是国内的 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码，将 gitee 源换成 github 源进行拉取代码。如果依然拉取不到代码，可以发邮件向我要代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin c73e4993fdc7fb8cdea2ab8367ba004003a94c27
```

获取代码之后，进入 Workbench/FacikereruyekoKiherbawjaiwha 文件夹，即可获取到源代码

更多技术博客，请参阅 [博客导航](https://blog.lindexi.com/post/%E5%8D%9A%E5%AE%A2%E5%AF%BC%E8%88%AA.html )
