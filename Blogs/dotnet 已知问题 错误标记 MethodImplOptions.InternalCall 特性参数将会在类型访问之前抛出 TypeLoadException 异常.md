本文将记录一个 dotnet 的已知问题。当自己不小心在方法上不正确标记了 MethodImplAttribute 特性时，错误选择了 MethodImplOptions.InternalCall 参数，那将会在运行的过程在，在此类型被访问之前就抛出了 System.TypeLoadException 异常，错误信息是 Internal call method with non_NULL RVA 内容

<!--more-->


<!-- 发布 -->
<!-- 博客 -->

遇到这个错误时，是比较难定位到具体的问题的。首先异常信息里面最多只是带上了类型名，没有告诉咱具体是哪个方法调用错误了。其次，异常的信息是 System.TypeLoadException 异常，且异常内容里面没有十分明确说明具体问题。不过有了 Internal call method 关键词倒是还能根据此找到问题

下面我将和大家演示一下错误在方法上标记了 MethodImplOptions.InternalCall 特性参数的行为，以下的代码可以在本文末尾找到下载方法

```csharp
using System.Runtime.CompilerServices;

var f1 = new F1();

Console.WriteLine("Hello, World!");

class F1
{
    public F1()
    {
        var f2 = new F2();
    }
}

class F2
{
    [MethodImplAttribute(MethodImplOptions.InternalCall)]
    public void Foo()
    {

    }
}
```

上面代码之所以需要定义两个类型，是因为这个异常是会在 F2 类型被访问到之前就抛出异常，这也就导致了更加难以调试。上面代码运行的时候，异常是抛在了进入 F1 类型的构造函数里面，如下图

<!-- ![](image/dotnet 已知问题 错误标记 MethodImplOptions.InternalCall 特性参数将会在类型访问之前抛出 TypeLoadException 异常/dotnet 已知问题 错误标记 MethodImplOptions.InternalCall 特性参数将会在类型访问之前抛出 TypeLoadException 异常0.png) -->

![](http://image.acmx.xyz/lindexi%2F202311191122288520.jpg)

这也就是导致了此问题更加难以调试的原因

在异常里面带上了 TypeName 属性，属性里面将会写明是 F2 类型出错，但是具体是哪个方法标记错了也没有更多的提示

我将此调试问题报告给 dotnet 官方，请看 https://github.com/dotnet/runtime/issues/94967

如果大家遇到了 System.TypeLoadException:“Internal call method with non_NULL RVA.” 异常，可以先看看这个异常里面的 TypeName 属性，确定是哪个类型出错了，然后再看看是否这个类型存在方法错误标记了 MethodImplOptions.InternalCall 特性参数导致运行失败

本文以上代码放在[github](https://github.com/lindexi/lindexi_gd/tree/96014b44231c7e6920abf4373521359705e8cb0c/GagageheLoqearrergi) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/96014b44231c7e6920abf4373521359705e8cb0c/GagageheLoqearrergi) 欢迎访问

可以通过如下方式获取本文的源代码，先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin 96014b44231c7e6920abf4373521359705e8cb0c
```

以上使用的是 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin 96014b44231c7e6920abf4373521359705e8cb0c
```

获取代码之后，进入 GagageheLoqearrergi 文件夹
