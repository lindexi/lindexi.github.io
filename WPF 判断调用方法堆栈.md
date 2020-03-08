# WPF 判断调用方法堆栈

最近遇到一个问题，经常有小伙伴在类A的构造里调用静态函数B，但是这时B依赖于A的初始化完成，于是就无限循环。所以我需要在判断小伙伴调用B时是否在A的构造方法里，如果是就给他异常。

本文告诉大家如何使用 StackTrace 获得调用堆栈，并且判断当前是否构造调用

<!--more-->
<!-- CreateTime:2018/8/10 19:16:53 -->


<!-- 标签：wpf,黑科技 -->

假设有方法 Foo ，如果需要判断 Foo 的调用有哪些，可以使用下面的代码

```csharp
        public void Foo()
        {
            var stackTrace = new StackTrace();
        }
```

使用`var n = stackTrace.FrameCount;`可以得到当前的栈有几个，最顶部就是最近调用。

例如调用是 `lindexi->A->csdn->Foo` 那么对应的栈就是下表

|序号|方法|
|--|--|
|3|lindexi|
|2|A|
|1|csdn|
|0|Foo|

如果要判断当前的调用是构造函数，那么需要知道，构造函数就是`.ctor` 那么使用下面的代码就可以判断

```csharp
            var stackTrace = new StackTrace();
            var n = stackTrace.FrameCount;
            for (int i = 0; i < n; i++)
            {
                //判断构造
                var cto = ".ctor";
                var f = stackTrace.GetFrame(i).GetMethod();
                if (f.Name.Equals(cto))
                {
                    Console.WriteLine("构造使用");
                }
                Console.WriteLine(stackTrace.GetFrame(i).GetMethod().Name);
            }

```

建议把上面的代码复制到一个项目，自己跑一下就知道了

如果还需要判断是指定类型的构造函数，那么需要使用下面的代码，下面代码判断是类型`GqpluGkmoanvp`的构造调用

```csharp
        public void Foo()
        {
            var stackTrace = new StackTrace();
            var n = stackTrace.FrameCount;
            for (int i = 0; i < n; i++)
            {
                //判断构造
                var cto = ".ctor";
                var f = stackTrace.GetFrame(i).GetMethod();
                if (f.Name.Equals(cto))
                {
                    var t = f.DeclaringType;
                    if (t.IsSubclassOf(typeof(GqpluGkmoanvp)) || t == typeof(GqpluGkmoanvp))
                    {
                        Console.WriteLine("构造使用");
                    }
                }
                Console.WriteLine(stackTrace.GetFrame(i).GetMethod().Name);
            }
        }
    }
```

实际使用`t.IsSubclassOf(typeof(GqpluGkmoanvp))`有些多余，但是写了也可以。因为如果写在构造，那么就会先调用基类的构造方法，所以已经会出现判断到基类的构造方法。

例如有下面的类，在构造方法调用`Foo`，那么调用堆栈就是 `Foo`-`A1`-`A`

```csharp
public class A1:A
{
    public A1()
    {
        Foo();
    }
}

public class A
{

}
```

下面是我封装的一个方法，用于判断当前调用是否在某个类里的某个方法

```csharp
         /// <summary>
        /// 查看调用这个方式是否在某个类的某个方法
        /// </summary>
        /// <param name="class"></param>
        /// <param name="method"></param>
        /// <returns></returns>
        public static bool CheckStackClassMethod(Type @class, string method)
        {
            var stackTrace = new StackTrace();
            var n = stackTrace.FrameCount;
            for (int i = 1; i < n; i++)
            {
                var f = stackTrace.GetFrame(i).GetMethod();
                if (f.Name.Equals(method))
                {
                    var t = f.DeclaringType;
                    if (t == @class || t.IsSubclassOf(@class) || (@class.IsInterface && @class.IsAssignableFrom(t)))
                    {
                        return true;
                    }
                }
            }
            return false;
        }
```

代码放在[WPF 判断调用方法堆栈](https://gitee.com/lindexi/codes/qigv3dt12js9ywoakpbu631 )

<script src='https://gitee.com/lindexi/codes/qigv3dt12js9ywoakpbu631/widget_preview?title=CheckStackClassMethod'></script>

因为传入的类型可能是接口，所以类型判断就需要加上接口继承的，所有代码已经给了大家。

使用这个方法，可以把判断是否在某个类的某个方法调用这个函数就可以修改为下面代码

```csharp
       public void Foo()
        {
            if (CheckStackClassMethod(typeof(GqpluGkmoanvp), ".d"))
            {
                Console.WriteLine("构造使用");
            }
        }
```

我经过测试下使用这个函数，判断是否我要保护的方法在我不想要被调用的类调用，如果是就抛出异常。接着我的朋友的代码跑不过，就把我祭天。

感谢 [walterlv](https://walterlv.oschina.io/ )

![](http://image.acmx.xyz/34fdad35-5dfe-a75b-2b4b-8c5e313038e2%2F2017101220537.jpg)

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。  