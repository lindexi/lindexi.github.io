
# WPF 已知问题 Popup 失焦后导致 ListBox 无法用 MouseWheel 滚动问题和解决方法

本文记录在 Popup 失焦后导致 ListBox 无法用 MouseWheel 滚动问题

<!--more-->


<!-- CreateTime:2022/3/10 12:09:08 -->

<!-- 发布 -->

原因：

Popup虽然是个完整独立的窗体，但它的激活要靠它的“父窗口”间接来激活，这里之所以说是“父窗口”，是因为它本身并没有真正的“父窗口”，它只是从“父窗口”里产生的一个游离的“子窗口”，也就是说它没记住它的“父亲”，但是它的“父亲”倒是记住它了，在“父亲”被激活的时候，“父亲”会去主动激活它这个不肖的“儿子”。

所以问题解决就从激活“父窗口”开始，在Popup窗体的PreviewMouseDown事件处理函数中，直接激活“父窗口”就可以了。写成伪代码如下：

```csharp
        popup.PreviewMouseDown += DialogPopup_PreviewMouseDown;

        private void DialogPopup_PreviewMouseDown(object sender, System.Windows.Input.MouseButtonEventArgs e)
        {

            this.Activate();//this为其理论上的父窗口，还请替换为你的实际代码。另外，根据代码规范，不要写 this. 哦

            this.Focus();
        }
```





<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。