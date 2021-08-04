
# dotnet 读 WPF 源代码笔记 提升调试效率的 NamedObject 类型

本文来聊聊 WPF 那些值得称赞的设计中的 NamedObject 类型。在 WPF 中，有很多值得我学习的设计开发思想，其中就包括本文将要介绍的 NamedObject 类型。此类型的定义仅仅只是为了方便调试，而没有具体的业务功能

<!--more-->


<!-- CreateTime:2021/8/2 8:31:44 -->


<!-- 标签：WPF，WPF源代码 -->
<!-- 发布 -->

在 WPF 的 WindowsBase 程序集里面就定义了 WPF 的很多基础类型，例如咱都在用的 DependencyObject 等类型。本文的 NamedObject 也正是定义在 WindowsBase 程序集里面众多的类型之一，代码就放在 `src\Microsoft.DotNet.Wpf\src\WindowsBase\MS\Internal\NamedObject.cs` 文件里面

这是一个代码量很少的类型，如下面的内容，就是 NamedObject 的全部代码

```csharp
    internal class NamedObject
    {
        public NamedObject(string name)
        {
            if (String.IsNullOrEmpty(name))
                throw new ArgumentNullException(name);

            _name = name;
        }

        public override string ToString()
        {
            if (_name[0] != '{')
            {
                // lazily add {} around the name, to avoid allocating a string
                // until it's actually needed
                _name = String.Format(CultureInfo.InvariantCulture, "{{{0}}}", _name);
            }

            return _name;
        }

        string _name;
    }
```

按照 dotnet 里面的常用设计，咱的 ToString 方法大多数都是用来调试使用，或者序列化使用。而 NamedObject 类型没有序列化的需求，因此就剩下一个调试的功能了。可以看到 NamedObject 没有任何的属性定义，也没有任何的方法。实际除了调试以外，就和 object 对象是一个功能。其实这是对的，这就是专门用来辅助调试的类型

为什么需要定义 NamedObject 类型来做调试辅助？原因是在 WPF 中，有某些地方的逻辑需要用到一个空的 Object 对象，而此对象不是用来做类里面的内部锁的信息，此空对象将会在框架层传输。在框架层传输一个空的对象无疑会让开发者在调试时感觉到无从下手，原因其实和空异常差不多。假定在某个业务逻辑里面，收到了其他模块发生过来的一个非预期的对象，刚好这个对象是一个空的 object 对象，此时请问这个空的 object 对象是什么，是由哪个模块创建的

为了构建出一个稳固的 UI 框架，或者为了方便后续像我这样水平一般的开发者可以参与到框架的开发，古老的 WPF 框架开发大佬们就定义了 NamedObject 对象。如名字一样，这个类型仅仅只是一个带命名的 object 对象而已。但一个带命名的 object 对象就相当于给代码加上了单位，可以极大提升框架开发调试遇到一个空对象时了解这是由哪个模块创建的

如 NamedObject 被 DependencyProperty 使用时的例子，在 DependencyProperty 里面，如果咱有某个未定义的依赖属性，或者说在绑定或属性转换器里面失败时返回一个未定义的属性时，按照最佳实践，咱应该返回 DependencyProperty 的 UnsetValue 属性

在其他业务端或 WPF 框架内收到了一个 Object 的时候，如何可以了解到这是 DependencyProperty 的 UnsetValue 属性，而不是开发者用户在业务层自己创建的某个对象？假定咱的 DependencyProperty 的 UnsetValue 属性采用空 object 对象，那么意味着调试时需要通过 VisualStudio 的创建对象 Id 的功能，通过给 DependencyProperty 的 UnsetValue 属性创建 Id 才能通过 Id 判断对象是否相同

无疑，如果只有一次两次需要如此调试，那还可以。如果每次调试框架时都需要执行如上步骤，通过 VisualStudio 的创建对象 Id 的功能才能进行调试，那小心键盘被 WPF 框架开发者砸了

古老的 WPF 框架开发大佬们给 DependencyProperty 的 UnsetValue 属性的定义如下

```csharp
   public static readonly object UnsetValue = new NamedObject("DependencyProperty.UnsetValue");
```

此时的优势在于当我拿到一个 object 对象的时候，可以在 VisualStudio 里面快速看到此对象是一个带命名的 `DependencyProperty.UnsetValue` 对象，此时就可以快速了解到此对象的创建者以及业务意图

值得我学习的是，不要轻易在对外公开的传递的对象，使用 object 对象，而是给此对象一个确切的定义类型。如果可以的话，再给这个确切的定义类型附加一句用来辅助调试的话，如 `"DependencyProperty.UnsetValue"` 这个字符串。这样可以方便在框架层进行调试时，了解传输的对象的创建者，以及开发者的意图

当前的 WPF 在 [https://github.com/dotnet/wpf](https://github.com/dotnet/wpf) 完全开源，使用友好的 MIT 协议，意味着允许任何人任何组织和企业任意处置，包括使用，复制，修改，合并，发表，分发，再授权，或者销售。在仓库里面包含了完全的构建逻辑，只需要本地的网络足够好（因为需要下载一堆构建工具），即可进行本地构建





<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。