
# WPF 设置窗口不跟随触摸惯性拖动抖动

默认在触摸滑动 ListView 等控件的时候，如果滑动到控件底部，会让 WPF 窗口也跟随触摸惯性滑动

<!--more-->


<!-- 发布 -->

解决方法是通过在 Window 里面重写 OnManipulationBoundaryFeedback 方法，进行禁用

```csharp
        protected override void OnManipulationBoundaryFeedback(ManipulationBoundaryFeedbackEventArgs e)
        {
            // 方法里面不需要写任何代码，就能解决这个问题
        }
```

详细请看

[wpf listbox touch 整个窗口移动 - 三叶草╮ - 博客园](https://www.cnblogs.com/luohengstudy/p/4139445.html )

[dotnet Framework 源代码 · ScrollViewer - 云+社区 - 腾讯云](https://cloud.tencent.com/developer/article/1518374 )





<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。