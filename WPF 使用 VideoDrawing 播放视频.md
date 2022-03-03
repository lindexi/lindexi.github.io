# WPF 使用 VideoDrawing 播放视频

本文告诉大家如何在 WPF 使用 VideoDrawing 进行视频播放

<!--more-->
<!-- CreateTime:2022/2/24 20:14:06 -->

<!-- 发布 -->

用这个方法有什么优势？其实只是想作为某个控件的背景，某个控件的背景使用视频而已

控件的背景使用 DrawingBrush 传入，在 DrawingBrush 传入 VideoDrawing 即可。创建 VideoDrawing 需要一个 MediaPlayer 和给定视频的宽度和高度

如以下代码，实现拖入一个视频文件，就作为背景进行播放。在 XAML 的代码如下

```xml
    <Grid Background="Transparent" AllowDrop="True" Drop="Grid_OnDrop">

    </Grid>
```

给 Grid 加上 Background 只是为了让 Grid 能收到拖入文件的事件而已，在 `Grid_OnDrop` 方法里面，加上拖入文件播放的逻辑

```csharp
        private MediaPlayer? MediaPlayer { set; get; }

        private void Grid_OnDrop(object sender, DragEventArgs e)
        {
            MediaPlayer?.Close();

            var fileList = (string[]?) e.Data.GetData(DataFormats.FileDrop);

            if (fileList is not null && fileList.Length > 0)
            {
                var mediaPlayer = MediaPlayer = new MediaPlayer();
                mediaPlayer.Open(new Uri(fileList[0]));

                var videoDrawing = new VideoDrawing()
                {
                    Player = mediaPlayer,
                    Rect = new Rect(new Size(Width, Height))
                };
                var drawingBrush = new DrawingBrush(videoDrawing);
                Background = drawingBrush;
                mediaPlayer.Play();
            }
        }
```

以上就是所有的代码

有哪些视频能播放？系统解码器能解的大部分的视频

可以使用上面的代码用来测试在 WPF 应用播放视频的性能哦，记得切换到 Release 发布版本，且不要在 VisualStudio 进行调试

本文所有代码放在[github](https://github.com/lindexi/lindexi_gd/tree/b3ff420fdce51e05d2c097a20145380766512fdb/ChairjuchiwhiRinehawwheago) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/b3ff420fdce51e05d2c097a20145380766512fdb/ChairjuchiwhiRinehawwheago) 欢迎访问

可以通过如下方式获取本文的源代码，先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin b3ff420fdce51e05d2c097a20145380766512fdb
```

以上使用的是 gitee 的源，如果 gitee 不能访问，请替换为 github 的源

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
```

获取代码之后，进入 ChairjuchiwhiRinehawwheago 文件夹

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
