#  win10 uwp MVVM 语义耦合

最近在我写的框架，小伙伴告诉我，可能有语义耦合，那么本文就来告诉大家，为什么会出现语言耦合

<!--more-->
<!-- CreateTime:2018/8/10 19:16:53 -->

<!-- csdn -->

之前我写了一个轻量的框架，参见[win10 uwp MVVM 轻量框架 ](https://lindexi.oschina.io/lindexi/post/win10-uwp-MVVM-%E8%BD%BB%E9%87%8F%E6%A1%86%E6%9E%B6.html )。在使用的过程，发现了这个框架可以让开发者写出语义耦合的代码。

在开始讲框架之前，先让我告诉大家，什么是语义耦合。

例如有一个框架，在框架的代码都没有任何的耦合，如 View 的界面和 ViewModel 是分开在两个工程，而且只有 View 引用 ViewModel  ，这样从静态的代码分析可以说，ViewModel 没有耦合 View 。但这是不是真的就没有耦合？实际上可能还是有语义的耦合。

![](http://image.acmx.xyz/34fdad35-5dfe-a75b-2b4b-8c5e313038e2%2F20182910446.jpg)

在小伙伴使用框架，但是对 MVVM 的理解不是很深的时候，就容易写出下面的代码

```csharp
ViewModel：

发送 刷新 列表消息

View 

处理 刷新列表消息
```

这时，因为 ViewModel 写出了刷新列表的消息，所以刷新列表这个界面相关的消息就耦合了界面。也就是在 ViewModel 是处理了部分的界面的逻辑。

在很多的代码，包括继龙大神的、于大神的代码，发现了在 ViewModel 使用了 Visibility 的属性，需要知道 Visibility 是界面的属性，如果界面绑定了 Visibility 那么 ViewModel 就需要对 Visibility 处理。这时的界面是需要写 ViewModel 的开发者知道的。这时界面修改的话，例如原来在条件下需要 Visibility 隐藏的代码就需要修改为不隐藏。这时修改界面就需要修改 ViewModel 。

如果在 ViewModel 耦合了界面的控制，那么为什么需要 ViewModel ？ 实际上的 ViewModel 是抽象页面，所以不能对 ViewModel 添加对界面控制的代码。在太子的博客[语义耦合](https://walterlv.github.io/post/semantic-coupling.html )已经有说到关于 Message 耦合的问题。同样也给出一些解决方法。

下面我来告诉大家一些比较好的处理方法。

在几乎所有主流的 MVVM 框架，都提供了 Message 给 ViewModel 和 View 通信。而且在我的框架用了 Message，所以对 Message 是比较难写的一点。如果限制很多，那么很多开发者就不会使用，解决这个框架不好用。如果给的功能太多，那么容易出现语义耦合，而且使用这个功能做框架设计没有提供的功能。

需要和大家分享一个故事，为什么 微软 把Action 的参数提供了 16 个？因为他需要考虑全世界所有的开发者。但是一般的开发者建议使用的参数只有两个，但是在一个足够通用的框架，是需要做到对少数的开发者也可以使用。所以不要认为只要框架提供了，就可以使用。一个好的开发者会去阅读框架的文档，然后使用框架开发者希望使用的方式开发。但是一个好的框架是开发者不需要去读文档，看到了框架就会使用。这是矛盾的，但是和框架开发者的能力有关。微软框架大部分都是很好的，但是也有一些代码写的很差。最近我在写高性能笔的时候就发现了他的代码的问题，已经帮他修复了，但是现在微软几乎不做 .net Framework 了，把他的很多代码都放在 .net core ，然后就经常看到有大神修改了算法，提高了性能。

回到问题，如何在开发中解决 MVVM 的语言耦合，实际上这不是一个技术问题。建议的方法是让开发者的代码经过审查，现在在开发的时候，所有的代码都需要提MR，在来源的开发中，也是需要提MR，这样就可以容易发现在代码中存在的语义耦合。现在通过工具是难以发现的。

如果发现了存在语义耦合，那么如何解决？

这个需要分析一下，一般做法是让具体的命名写为抽象。如上面的代码，从ViewModel 告诉 View 刷新列表，为什么需要 ViewModel 知道 View 需要刷新列表，他可以使用一个抽象的命名，例如告诉 View 现在更新了数据。于是 View 根据ViewModel 的消息进行刷新列表，这样就不会出现 ViewModel 的语言耦合。

但是很多的代码都可以使用状态来获得刷新和修改，所以这时就不需要使用消息。

另一个建议是最好不要在 ViewModel 定义界面的控制的方法，例如 Visibility ，不可以让ViewModel 去控制 View 。在微软提出的就说到，ViewModel 是驱动数据，所以 ViewModel 只是转发数据，这样才可以减少耦合。

一个项目使用了框架会比不使用框架的可维护要好很多，即使使用的时候存在一些问题，但是也比不使用好。

参见：[语义耦合（Semantic Coupling） - walterlv](https://walterlv.github.io/post/semantic-coupling.html )

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。 
