
# Win10 UWP Intro to controls and events


<!--more-->



<div id="toc"></div>

这篇翻译，如果有不对可以发邮箱

为创建页面，可以通过按钮，TextBox输入，组合框来显示数据，获得用户输入。添加一个控件可以使用三个关键步骤：

- 添加一个控件到界面

- 设置控件属性，高度，宽度，颜色

- 添加控件的事件

## 添加控件


可以使用以下任意方式添加控件

- 使用界面直接拖控件，Blend直接在工具箱把控件拖到界面
- 
![](http://i12.tietuku.cn/5af30d67c9536b70.png)

 点击Button，拖动Button界面
 ![](http://i13.tietuku.cn/d1ce83f232b174fb.jpg)
 
-  用Xaml编辑`<Button Content="确定"></Button>`
   
- 代码添加控件

在visual studio可以使用工具箱、Xaml编辑器、设计器，属性窗口

- 工具箱

![](http://i13.tietuku.cn/75399cc4641b063e.png)


- Xaml编辑器

![这里写图片描述](http://img.blog.csdn.net/20160323170242683)

- 设计器

![这里写图片描述](http://img.blog.csdn.net/20160323170453139)

- 属性窗口

![这里写图片描述](http://img.blog.csdn.net/20160323170538139)

工具箱显示很多可以用在软件的控件，可以拖动控件到界面，可以双击控件，控件就会自动添加到软件。

双击TextBox

```xml
<TextBox x:Name="textBox" TextWrapping="Wrap" Text="TextBox"/>
```

## 命名控件

为了在代码改变控件，可以给控件名字，`x:Name`后面写控件名称，控件名称不能重复，不能数字开头

可以使用属性来命名控件

点击控件，在属性可以看到
![这里写图片描述](http://img.blog.csdn.net/20160323171006786)

在名称写上控件名

## 设置控件属性

可以在属性选择控件属性

![这里写图片描述](http://img.blog.csdn.net/20160323171006786)

可以编辑Xaml写控件

![这里写图片描述](http://img.blog.csdn.net/20160323171135819)

如果你设置了一个你不要，可以重设属性

![](http://i12.tietuku.cn/54149a3630910083.png)
点击重新设置

设置颜色可以使用下面的颜色表

![](http://i12.tietuku.cn/8f87d0190a7279e7.png)

在Xaml写Visual studio在你按下一个键就会提示

![](http://i12.tietuku.cn/3ce2d0efe79ef44f.png)

## 控件事件

每个控件都有很多事件，可以使用Xaml，属性创建事件，创建事件的方法是事件处理，参见：https://msdn.microsoft.com/windows/uwp/xaml-platform/events-and-routed-events-overview

创建事件可以在属性

![](http://i12.tietuku.cn/7ddf9aae007a821a.png)
选择事件，写名称，按回车，就会到cs，事件处理第一个参数是发送者，引用对象，第二个是事件数据

我们创建一个Click

```csharp
        private void Button_Click(object sender, Windows.UI.Xaml.RoutedEventArgs e)
        {
            Button button=sender as Button;//sender 发送者
        }
```
如果有给按钮名称，可以在代码

![](http://i12.tietuku.cn/2cc32155efa5e434.png)

原文：https://msdn.microsoft.com/windows/uwp/controls-and-patterns/controls-and-events-intro





<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。