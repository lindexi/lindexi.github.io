# win10 UWP ListView 

<!--more-->

## 横向布局

默认 ListView 是垂直，那么如何让 ListView 水平？

可以使用下面代码

```csharp
            <ListView.ItemsPanel>
                <ItemsPanelTemplate>
                    <StackPanel Orientation="Horizontal"></StackPanel>
                </ItemsPanelTemplate>
            </ListView.ItemsPanel>
```

设置代码可以进行横向。

如果发现 UWP ListView 横向没有滚动条，可以使用 ScrollViewer 添加


```csharp
            <ListView  ScrollViewer.VerticalScrollBarVisibility="Disabled"  
                       ScrollViewer.HorizontalScrollBarVisibility="Auto"
                       ScrollViewer.HorizontalScrollMode="Enabled"                  
                       ScrollViewer.VerticalScrollMode="Disabled">
```

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://i.creativecommons.org/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。