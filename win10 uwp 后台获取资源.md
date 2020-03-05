# win10 uwp 后台获取资源

本文告诉大家，从后台代码获取界面定义的资源。

<!--more-->
<!-- CreateTime:2018/8/10 19:17:19 -->


如果一个资源是写在 App 的资源，那么如何使用代码去获得他？

简单的方法是使用下面的代码

```csharp
Application.Current.Resources["Key"]

```

其中 Key 就是资源的 Key ，这样就可以从后台获取资源。

需要知道的是，获取的资源类型是 Object ，这时，建议使用 cast 转换，而不是使用 as。

原因就是使用的方式就是这时是否知道资源的类型，一旦写出资源的 Key ，那么就是确定了这个类型，所以从逻辑上，这时就是知道他的类型，知道类型的转换，就使用 cast 的方法。cast 的方法指使用括号的方法强转。

如果是在 xaml 使用，请直接使用 staticResource 的方法就好了。

资源的定义是靠近使用的地方优先。

如果有多个资源使用相同的 Key ，那么哪个资源靠近使用的地方，就是使用哪个资源。可以自己尝试写个呆磨试试，看起来很容易做的样子。如果一个资源写在 App.xaml 那么整个程序都可以用，而且这个资源不会被释放。所以如果想在 App.xaml 使用资源，请小心。如果资源太多，会让软件的启动速度变慢。另外，请不要直接把资源写在 App.xaml ，建议是写在一个资源文件，然后在 App.xaml 使用 Merge 的方式。

参见：http://blog.csdn.net/fwj380891124/article/details/8153229

![](http://image.acmx.xyz/34fdad35-5dfe-a75b-2b4b-8c5e313038e2%2F20178885742.jpg)

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。  
 