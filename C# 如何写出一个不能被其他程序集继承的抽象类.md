# C# 如何写出一个不能被其他程序集继承的抽象类

我需要限定某个抽象类只能在我程序集类实现，而不支持其他程序集实现，也就是我需要一个不能被继承的抽象类

<!--more-->
<!-- CreateTime:2019/8/31 16:55:58 -->


在 C# 里面有抽象类和接口，这两个都是期望被继承才能被使用，而抽象类是可以做到只能在自己程序集和程序集可见的其他程序集实现，而在其他的程序集是不能实现

在开始告诉大家如何写之前，需要说明在什么时候需要使用这个方法

有一些接口或方法等需要传入一个抽象的类，但这个抽象类里面有很多方法或属性的定义是和程序集内逻辑相关的，也就是开发者如果直接在外面继承这个抽象类实现出来的一定是不符合预期的。此时就需要用到本文的方法

此时就不能使用接口，因为接口是无法限制只能在程序集内实现，也就是在程序集外依然可以用户自己定义

写出一个只能在程序集内继承的抽象类的方法是在抽象类里面放一个需要被重写的部件，这个部件的访问优先级为程序集内，例如下面的代码，在抽象类 A 里面添加了需要继承重写的 B 方法，而 B 需要用到程序集内才能访问的接口

```csharp
// 程序集 1
    public abstract class A
    {
        internal abstract IB B();
    }

    internal interface IB
    {

    }
```

上面代码的抽象类 A 就只能在程序集 1 中实现，而无法在程序集 2 重写，例如下面代码将会提示重写方法 B 的接口没有权限，而不重写方法 B 就不能继承 A 于是开发者就不能自己实现继承，这样的写法可以让 SDK 开发了多态，但又不让开发者传入不对的值

```csharp
// 程序集 2
    class C : A
    {
        internal override IB B()
        {
            return null;
        }
    }
```

这样的写法在框架里面有很多，例如 WPF 框架里面的 Brush 画刷就是这样做的，请看 [Brush.cs](https://github.com/dotnet/wpf/blob/ae1790531c3b993b56eba8b1f0dd395a3ed7de75/src/Microsoft.DotNet.Wpf/src/PresentationCore/System/Windows/Media/Brush.cs ) 的实现

在 WPF 里面认为画刷资源是不能给开发者自己写的，因为开发者应该不能了解画刷是如何做的，而在很多类的属性或方法参数都需要传入画刷，因为画刷有纯色画刷等，不能写统一的画刷，于是就通过抽象类，用本文的方法做到让开发者只能用程序集里面定义的画刷。这样可以让框架代码做到支持多个不同的画刷同时让开发者不会随意继承画刷，解决在框架里面大量参数判断

本文用的代码放在github欢迎大家下载

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
