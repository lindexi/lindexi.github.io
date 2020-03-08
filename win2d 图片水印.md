# win2d 图片水印

本文告诉大家如何使用 win2d 给图片加上水印。

<!--more-->
<!-- CreateTime:2018/12/25 10:37:52 -->


<div id="toc"></div>

<!-- 标签：水印，win2d，uwp,渲染 -->

## 安装

首先需要使用 Nuget 安装 win2d ，安装参见[win10 uwp win2d 入门 看这一篇就够了](https://lindexi.gitee.io/post/win10-uwp-win2d-%E5%85%A5%E9%97%A8-%E7%9C%8B%E8%BF%99%E4%B8%80%E7%AF%87%E5%B0%B1%E5%A4%9F%E4%BA%86.html )

如果没有更新 dot net core 那么在运行可能会出现下面异常`System.TypeLoadException: Requested Windows Runtime type 'Microsoft.Graphics.Canvas.Text.CanvasTextLayout' is not registered`

那么直接更新 dot net core 到最新，然后清理项目就可以

## 获得图片

要对图片处理，首先需要拿到图片，拿到图片的方法可以是从[剪贴板](https://blog.csdn.net/lindexi_gd/article/details/50479180 )获得或者使用文件选取拿到。

如果是从剪贴板拿到图片，需要把图片保存到本地的临时文件夹，然后拿到文件。

如果使用文件选取拿到文件，可以使用这个方法

```csharp
          var pick = new FileOpenPicker();
            pick.FileTypeFilter.Add(".jpg");
            pick.FileTypeFilter.Add(".png");

            var file = await pick.PickSingleFileAsync();
```

注意后缀名用的是 `.`+后缀名，这里我写的是很少的图片后缀名，实际上可以支持的图片后缀是很多。

## 创建图片

如果需要对图片处理，使用的是 CanvasRenderTarget ，可以看到这个类需要传入两个参数`ICanvasResourceCreatorWithDpi`，`Size`，我也就使用这个函数

在 win2d 使用图片需要 CanvasBitmap ，这个类不可以直接创建，需要通过`LoadAsync`、`CreateFromBytes`、`CreateFromColors`、`CreateFromSoftwareBitmap` 这些方法来创建，下面就使用第一个方法创建。

第一个方法有很多重载，需要注意，如果不是解决方案里的文件，千万不要使用文件名或 URI 的方法，因为经常出现文件无法访问。

> 如果不是解决方案里的文件，千万不要使用 fileName 或 URI 的方法读取图片，因为一般的文件是没有权限。即使使用 FilePick 拿到文件，文件的路径也可能拿不到。

建议使用的方法是使用流的重载，在上面，已经拿到文件，这时把文件读出来，传入就可以

```csharp
var duvDbecdgiu =
                await CanvasBitmap.LoadAsync(new CanvasDevice(true), await _file.OpenAsync(FileAccessMode.Read));
```

## 处理图片

现在创建 CanvasRenderTarget 处理图片，在使用 CanvasRenderTarget 记得释放，所以一般需要使用下面代码

```csharp
 using (var canvasRenderTarget = new CanvasRenderTarget(duvDbecdgiu, duvDbecdgiu.Size))
```

创建一个图片处理，大小就和图片大小相同。

在图片添加文字的方法实际上和在 win2d 的其他处理相同，具体可以去看我的win2d博客。

```csharp
                using (var dc = canvasRenderTarget.CreateDrawingSession())
                {
                    dc.DrawImage(duvDbecdgiu);
                    dc.DrawText("lindexi",
                        new Vector2((float) (duvDbecdgiu.Size.Width / 2), (float) duvDbecdgiu.Size.Height/2), Colors.Black);
                }
```

也许大家会觉得上面的`DrawImage`是做什么的，刚才不是从图片创建的？实际上从图片创建，但是没有画图片，也就是在使用的时候需要先画图片，然后画出文字。

## 保存

现在尝试保存一个图片，保存需要让用户选一个文件

```csharp
        var pick = new FileSavePicker();
                pick.FileTypeChoices.Add("image", new List<string>() {".jpg"});

                var file = await pick.PickSaveFileAsync();
```

保存很简单

```csharp
await canvasRenderTarget.SaveAsync(await file.OpenAsync(FileAccessMode.ReadWrite),CanvasBitmapFileFormat.Jpeg);
```

注意保存的格式可以是很多，但是后缀名需要和保存的格式相同。

现在这个功能写在图床

![](http://image.acmx.xyz/65fb6078-c169-4ce3-cdd9-e35752d07be0%2F2018318182752.jpg)

[win10 uwp win2d 入门 看这一篇就够了](https://lindexi.gitee.io/post/win10-uwp-win2d-%E5%85%A5%E9%97%A8-%E7%9C%8B%E8%BF%99%E4%B8%80%E7%AF%87%E5%B0%B1%E5%A4%9F%E4%BA%86.html )

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。