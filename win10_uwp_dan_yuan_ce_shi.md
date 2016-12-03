# win10 UWP 单元测试

我们在写代码的时候不能保证我们写出来的代码是正确的，所以我们经常要单元测试。
<!--more-->

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

```
using Microsoft.VisualStudio.TestPlatform.UnitTestFramework;
```

在类定义前

```
[TestClass]
```

在类里面加函数 clipboard_substitution
函数需要在函数前

```
 [TestMethod]
```

我的函数需要测试输入一个文本是否会把选择的string替换输入文本

我们在测试单元写测试输入

```
            var view =new produproperty.ViewModel.winmain(null);
            string text = "要替换文本";
            //把替换两个字替换为string
            view.text = text;
            view.select = 1;
            view.select_length = 2;
            view.clipboard_substitution("string");
```
然后写Assert

```
            Assert.AreEqual("要string文本",view.text);
```
右击运行
![这里写图片描述](http://img.blog.csdn.net/20160221135932877)

可以在运行 所有测试
![这里写图片描述](http://img.blog.csdn.net/20160221140009378)

测试通过
![这里写图片描述](http://img.blog.csdn.net/20160221140153722)

我们还要做一些诡异测试

出现错误Index and length must refer to a location within the string.

这样就是我们函数有问题

如果通过了我们才可以说我们代码可以提交

Assert是返回结果true方法是测试通过，如果是其他就不通过，Assert可以有方法

|方法|描述|
|--|--|
|AreEqual|两个值是否相等|
|AreNotEqual|两个值不相等|
|AreNotSame|两个值不相同|
|AreSame|两个值相同|


<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://i.creativecommons.org/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。


