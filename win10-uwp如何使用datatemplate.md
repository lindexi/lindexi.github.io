#win10 uwp如何使用DataTemplate

【】

这是数据模板，一般用在数组的绑定，显示数组中的元素。

假如我们有一个列表，列表里是书，包括书名、作者、还有出版，那么我们只有源信息，如何把它显示到我们的ListView，就需要DataTemplate。

使用很简单，我们可以定义在资源，也可以定义在ItemTemplate。

数据模板有绑定的问题。

我们使用`Binding`和WPF其实没有多少不同，在Mode只有`OneWay`,`OneTime`,`TwoWay`。我们使用的`x:bind`在DataTemplate才和原来有一些不同。

我们使用`x:bind`需要我们对我们数据的类型，这个在前没有，我开始不知，弄了好久，最后才知道，还有一个，UWP默认是OneTime，也就是绑定只有一次。


我们假如我们的类型是放在Model，我们需要在开始，就是页面写我们类的命名空间





##转换

UWP的Convert和WPF差不多。

如果我们的数据需要转换，转换需要我们的变量，在我上次使用win10 uwp 进度条使用了是静态的数值，这样不好，那么我们需要做一个简单使用我们类数据转换器，在我们常用的是把它写staticResource

参见：http://www.cnblogs.com/horan/archive/2012/02/27/2368262.html

UWP

##UWP 默认应用打开文件

##UWP 文件md5

##UWP协议


 <a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://i.creativecommons.org/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。

