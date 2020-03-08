# WPF 列表自动换行

本文告诉大家如何在 ListView 或 ListBox 使用 WrapPanel 让里面的控件自动换行

<!--more-->
<!-- CreateTime:2019/2/11 8:55:31 -->

<!-- csdn -->

在 WPF 可以通过修改 ItemsPanel 设置使用不同的 ItemsPanel 如 StackPanel 等

可以通过使用 WrapPanel 让控件，如果宽度在 ListView 或 ListBox 之外就换行

```csharp
            <ListView.ItemsPanel>
                <ItemsPanelTemplate>
                    <WrapPanel Orientation="Horizontal"></WrapPanel>
                </ItemsPanelTemplate>
            </ListView.ItemsPanel>
```

如果现在就运行代码，看到所有的控件都放在水平，没有在 ListView 自动换行

因为没有设置 ListView 不能水平移动，在 ListView 的宽度是无限，设置 ListView 的框架就是他的可见宽度可以使用下面代码

```csharp
ScrollViewer.HorizontalScrollBarVisibility="Disabled"
```

现在的 ListView 看起来大概是下面代码

```csharp
<ListView ScrollViewer.HorizontalScrollBarVisibility="Disabled">
  <ListView.ItemsPanel>
    <ItemsPanelTemplate>
      <WrapPanel Orientation="Horizontal" />
    </ItemsPanelTemplate>
  </ListView.ItemsPanel>
  ...
</ListView>
```

<!-- ![](image/WPF 列表自动换行/WPF 列表自动换行0.png) -->

![](https://i.loli.net/2019/02/09/5c5ec21f5336b.png)

[win10 UWP ListView](https://lindexi.gitee.io/post/win10-UWP-ListView.html)

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
