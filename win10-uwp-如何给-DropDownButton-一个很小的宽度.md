
# win10 uwp 如何给 DropDownButton 一个很小的宽度

在 UWP 的 Microsoft.UI.Xaml 提供了一个带下箭头的按钮，这就是 DropDownButton 这个按钮继承 Button 按钮，基本表现相同，但是如果给这个按钮一个很小的宽度，将会看不到下箭头图片

<!--more-->


<!-- 发布 -->

原因是如果最小宽度那么下箭头将没有足够空间显示，虽然左边依然有空白地方，但是空白地方有最小宽度要求

解决方法是通过 Padding 属性，让整个按钮的内容移动，让空白地方移动到按钮外，让下箭头移动到可以显示的地方

```xml
<DropDownButton Margin="10,10,10,10" Width="17" Height="30" Padding="-15,0,0,0"></DropDownButton>
```

上面代码核心就是 `Padding="-15,0,0,0"` 通过 Padding 可以设置按钮的左上右下各个内容边距的值

现在看起来的效果如下图

![](https://i.stack.imgur.com/uSccD.png)

更多关于 DropDownButton 请看 [DropDownButton Class - Windows UWP applications](https://docs.microsoft.com/en-us/uwp/api/microsoft.ui.xaml.controls.dropdownbutton?view=winui-2.3 )

这是在堆栈网小伙伴问的问题，请看 [c# - Change the width of DropDownButton in UWP - Stack Overflow](https://stackoverflow.com/a/60612482/6116637 )

本文源代码放在[github](https://github.com/lindexi/lindexi_gd/tree/15af922b55e564c853842238be4a682f66b6fe6f/LeceaberheafeKeacafiwhajaibaiwhi) 欢迎小伙伴访问





<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。