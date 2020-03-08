# WPF 图片移除视觉树内存泄漏

本文告诉大家一个已知问题，在保存图片元素对象时，如果在图片移除视觉树之后再设置图片源为空，那么原有的图片源依然被图片元素引用不会释放

<!--more-->
<!-- CreateTime:2020/1/7 14:25:52 -->

<!-- 发布 -->

如写一个按钮，在点击事件里面创建 RenderTargetBitmap 加入到新建的图片元素，然后在下次点击事件时，将图片元素从视觉树移除之后设置图片源为空

```csharp
        private void Button_Click(object sender, RoutedEventArgs e)
        {
            // 每次点击此按钮会将当前呈现的图片移除视觉树，再将其Source属性设置为null。
            // 然后新建一个Image控件，并将其Source属性设置为RenderTargetBitmap对象，再呈现出来。
            // 再次过程中，RenderTargetBitmap对象从来不会被回收，造成内存泄露。
            // 可以从资源管理其中观察到程序的内存持续上涨的现象。

            // Remove the current Image control from the  visual tree and set source is null when click button.
            // Then new a image control and add source to the RenderTargetBitmap object and show it.
            // You can see the gc never delete the RenderTargetBitmap object that make  memory leak.


            var oldBorder = RootGrid.Children.OfType<Border>().LastOrDefault();
            if (oldBorder != null)
            {
                var oldImage = (Image)oldBorder.Child;


                // 如果在Image控件移除视觉树之前将其Source属性设为null，并调用UpdateLayout方法。
                // 则RenderTargetBitmap对象可被回收，不会导致内存泄露。
                // 取消注释下面的代码可以观察到上述现象。
                // In order to solve it , you should set the image.Source is null and use UpdateLayout.
                // The below code can solve it.
                // oldImage.Source = null;
                // oldImage.UpdateLayout();

                // 将当前的Image控件移除视觉树。
                // Remove the current Image control from the  visual tree.
                RootGrid.Children.Remove(oldBorder);
                oldImage.Source = null;
                Borders.Add(oldBorder);
            }

            var bitmap = new RenderTargetBitmap(1024, 1024, 96, 96, PixelFormats.Default);

            var image = new Image { Source = bitmap };
            var border = new Border { Child = image };
            RootGrid.Children.Add(border);

            // 为了便于观察内存的变化，每次操作后都会进行垃圾回收。
            // In order to facilitate changes in memory, after each operation will be garbage collection
            GC.Collect();
        }

        public static readonly List<Border> Borders = new List<Border>();
```

上面代码本身的 Image 元素就是内存泄漏的，因为 Image 元素被 Border 引用，加入到静态数组

但是 RenderTargetBitmap 也内存泄漏，虽然在图片移除视觉树之后设置 `oldImage.Source = null;` 也就是从代码上没有任何对象引用 RenderTargetBitmap 类，但是此类还是内存泄漏了

解决方法是在移除视觉树之前设置为空，同时调用 UpdateLayout 方法，或者在下一次 Dispatcher 将图片移除视觉树

```csharp
     oldImage.Source = null;
     oldImage.UpdateLayout();
```

我在 [github](https://github.com/dotnet/wpf/issues/2397) 给微软报了这个问题，求点赞

[Known issus: WPF Image memory leak when remove image from visual tree · Issue #2397 · dotnet/wpf](https://github.com/dotnet/wpf/issues/2397 )

为什么会出现内存泄漏？原因是在图片继承的 UIElement 的布局方法会调用 OnRender 方法，而图片通过 DrawContext 的方式绘制了 Source 但是这个 DrawContext 的上下文被 UIElement 保存到 `_drawingContent` 字段

因为在调用 DrawContext 绘制图片时，将图片转换为MIL资源存放在 RenderData 类，而绘制完成之后将对应的值放在 `_drawingContent` 字段，也就是在 `_drawingContent` 引用了图片资源

此时设置图片的源为空，如果图片还在视觉树上，那么将会再次触发 OnRender 方法，在 OnRender 方法里面将会更新 RenderData 对图片源的引用

但是如果图片是被移除视觉树之后设置图片的源为空，那么不会再次触发 OnRender 方法，这样在 RenderData 存在对图片源的引用，此时将不会释放内存。如果在设置图片的源为空，然后不等待 OnRender 方法执行就将图片移除视觉树也是会内存泄漏。所以需要设置图片的源为空，然后调用 UpdateLayout 方法执行 OnRender 方法

其实这个内存泄漏问题很小，原因是如果 Image 元素对象没有被引用，那么图片就可以被释放，此时图片的源也可以释放

但是如果是一个大的做虚拟化的列表，此时在不可见的图片设置源为空，同时移除视觉树，此时图片的对象依然引用，虽然从代码上没有对图片源的引用，但是图片源依然在内存。也就是这个问题需要在做虚拟化列表时，注意对图片的移除视觉树

现在 WPF 开源了，有很多问题都可以从底层修改，欢迎大家关注[WPF官方开源仓库](https://github.com/dotnet/wpf ) 欢迎组队格式代码

其实我没有在本地编译成功 WPF 项目，所以干的最多的只是格式代码

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
