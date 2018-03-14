
# win10 uwp 初始屏幕

用户打开程序看到的第一个界面就是初始屏幕，UWP提供的是给图片，图片不好看，想要一个图片全屏，刚好是他给的尺寸，很难。我们可以快速在启动的时候，跳转到一个页面，这个页面就是做启动动画。
我们程序启动的时候需要加载数据，在加载数据的时候，还是显示开始的启动图片，看起来不好。本文主要说如何做一个启动界面类似段子之家的东西。

<!--more-->



<div id="toc"></div>
<!-- csdn -->

其实做这个很简单，我们需要在一开始就导航到启动页面。

我们需要去View新建一个页面，当然我把这个页面叫SplashPage，你可以使用你喜欢的名字。

修改导航，我是直接修改`App.xaml.cs`




<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。