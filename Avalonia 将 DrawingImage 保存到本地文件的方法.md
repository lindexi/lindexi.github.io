# Avalonia 将 DrawingImage 保存到本地文件的方法

本文将和大家演示如何在 11.2.0 的 Avalonia 框架上，将 Avalonia.Media.DrawingImage 保存存放为本地图片文件的方法

<!--more-->
<!-- CreateTime:2025/04/18 07:17:14 -->

<!-- 发布 -->
<!-- 博客 -->

核心需要借助 Avalonia.Media.Imaging.RenderTargetBitmap 类型，从 RenderTargetBitmap 创建 DrawingContext 对象，然后调用 DrawingContext 的 DrawImage 方法将 DrawingImage 绘制进入到 RenderTargetBitmap 里面，最后再调用 RenderTargetBitmap 的 Save 方法保存到本地文件

核心代码如下，假定已经拿到了 `Avalonia.Media.DrawingImage drawingImage` 对象

```csharp
            var imageSize = drawingImage.Size;
            using var renderTargetBitmap = new RenderTargetBitmap(new PixelSize((int) imageSize.Width, (int) imageSize.Height));

            using (DrawingContext drawingContext = renderTargetBitmap.CreateDrawingContext())
            {
                drawingContext.DrawImage(drawingImage, new Rect(new Point(), imageSize));
            }
           
            renderTargetBitmap.Save("1.png");
```

通过如上代码即可将 DrawingImage 存放到本地 1.png 文件

以上方法比调用 RenderTargetBitmap 的 Render 渲染临时创建的 Image 控件更好，因为以上方式开销更小且不受样式等的影响

```csharp
            // 不推荐的方法
            Image image = new Image
            {
                Source = drawingImage,
                Width = imageSize.Width,
                Height = imageSize.Height
            };
            //image.UpdateLayout(); 调用 UpdateLayout 方法是无效的，因为 Image 没有加入视觉树
            image.Measure(new Size(imageSize.Width, imageSize.Height));
            image.Arrange(new Rect(new Point(), imageSize));

            renderTargetBitmap.Render(image);
```

本文代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/25d5e60822d0e6b8036856b9e6e2c1329b1e5f54/AvaloniaIDemo/RerneyawyayWilalawha) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/blob/25d5e60822d0e6b8036856b9e6e2c1329b1e5f54/AvaloniaIDemo/RerneyawyayWilalawha) 上，可以使用如下命令行拉取代码。我整个代码仓库比较庞大，使用以下命令行可以进行部分拉取，拉取速度比较快

先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin 25d5e60822d0e6b8036856b9e6e2c1329b1e5f54
```

以上使用的是国内的 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码，将 gitee 源换成 github 源进行拉取代码。如果依然拉取不到代码，可以发邮件向我要代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin 25d5e60822d0e6b8036856b9e6e2c1329b1e5f54
```

获取代码之后，进入 AvaloniaIDemo/RerneyawyayWilalawha 文件夹，即可获取到源代码

更多技术博客，请参阅 [博客导航](https://blog.lindexi.com/post/%E5%8D%9A%E5%AE%A2%E5%AF%BC%E8%88%AA.html )