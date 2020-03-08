# win10 UWP 单元测试

我们在写代码的时候不能保证我们写出来的代码是正确的，所以我们经常要单元测试。
<!--more-->
<!-- CreateTime:2018/2/13 17:23:03 -->


<div id="toc"></div>

单元测试和重构都是在做完一个小小函数一般就要进行一次，越早做就越好，可以比较早发现问题，这时我们还记得我们写的内容，不过比重构好的是，重构我们经常不知道要叫什么名字，而单元测试反而就比较简单。

右击解决方案，添加新项目
![这里写图片描述](http://img.blog.csdn.net/20160221134353215)
C#->Windows->通用->单元测试应用
![这里写图片描述](http://img.blog.csdn.net/20160221134516798)

命名我是叫 测试

在新建单元测试右击引用
![这里写图片描述](http://img.blog.csdn.net/20160221134604919)

把工程引用

![这里写图片描述](http://img.blog.csdn.net/20160221134652691)

打开测试项目

一般测试哪个类我就会新建一个类名称和要测试类相同，类里面函数和要测试函数名相同。

我在做一个windows Markdown，里面有函数把剪贴的文本覆盖Textbox选文本，我不知道这个函数写的是不是对，于是我就在单元测试，新建一个类

测试函数所在的类是winmain，所以在单元测试新建一个类winmain

在新建类加上

```csharp
using Microsoft.VisualStudio.TestPlatform.UnitTestFramework;
```

最新版 是 `using Microsoft.VisualStudio.TestTools.UnitTesting;`

在类定义前添加特性，如下：

```csharp
[TestClass]
```

只有在类添加这个特性，这个类才可以做测试。

在类里面加函数 clipboard_substitution
函数需要在函数前添加特性，如下代码。添加了特性就是告诉vs，我就是测试函数，你可以运行我。

```csharp
 [TestMethod]
```

可以看到添加了这个，在左边就出现了一个按钮，可以进行测试，点击就会运行这个函数。

接下来就来进行测试一个小东西。

我的函数需要测试输入一个文本是否会把选择的 string 替换输入文本的小函数。

我们在测试单元写测试输入下面代码，如何写测试的请去看下 测试代码如何写 相关的文章，也就是把所有可能的输入和想要的输出写出来，判断是不是程序运行和要的一样，如果不是的话，报错。

```csharp
            var view =new produproperty.ViewModel.winmain(null);
            string text = "要替换文本";
            //把替换两个字替换为string
            view.text = text;
            view.select = 1;
            view.select_length = 2;
            view.clipboard_substitution("string");
```
上面的代码就是谢输入是什么，然后就是写输出是什么，判断程序的运行是否和想要的一样。判断是否一样，可以使用 Assert ，现在输入已经写完了，
然后写 Assert

```csharp
            Assert.AreEqual("要string文本",view.text);
```

看起来函数已经写完了，开始测试

右击运行
![这里写图片描述](http://img.blog.csdn.net/20160221135932877)

如果有很多个测试的函数，不需要一个个来，可以使用下面的方法执行测试类所有方法

可以在运行 所有测试
![这里写图片描述](http://img.blog.csdn.net/20160221140009378)

如果看到下面的图，那么
测试通过
![这里写图片描述](http://img.blog.csdn.net/20160221140153722)

我们还要做一些诡异测试，也就是程序考虑不到的，如
出现错误 Index and length must refer to a location within the string.

这样就是我们函数有问题，测试就是保证程序是正确的，也就是在正常的输入是正确的，对不特殊输入还可以做一些可以把特殊的输入变为正常的，或者其他的，就是不让程序直接就异常了。

当然有些输入还是需要异常的，于是异常，也可以测试。只需要在特性加一个希望的异常，这样就好了。

对于测试写完，
如果通过了我们才可以说我们代码可以提交

Assert 是返回结果 true 方法是测试通过，如果是其他就不通过，Assert 可以有方法推荐使用

|方法|描述|
|--|--|
|AreEqual|两个值是否相等|
|AreNotEqual|两个值不相等|
|AreNotSame|两个值不相同|
|AreSame|两个值相同|


## WPF 单元测试

对于 WPF 的单元测试，可以新建一个控制台项目，然后右击引用

![](https://ooo.0o0.ooo/2017/02/07/58998e2e7d476.jpg)

添加 Microsoft.VisualStudio.QualityTools.UnitTestFramework 然后其他和UWP一样。

注意命名空间 `using Microsoft.VisualStudio.TestTools.UnitTesting;`

然后把测试类写公开，其他和 UWP 一样


## 异步测试

参见：https://msdn.microsoft.com/zh-cn/magazine/dn818493.aspx


<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。




