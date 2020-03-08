# win10 uwp 使用油墨输入

现在很多人还是使用笔和纸来记录，那么可以在电脑输入方式和之前使用的方式一样，很多用户觉得会方便。在win10 我们有一个简单的方法去让用户输入，`InkCanvas`。现在edge，OneNote这些都有使用`InkCanvas`,我们可以在我们的手机上手写，我们也可以在我们电脑上用鼠标写，然后我们可以把我们写的保存图片，可以识别文字。

<!--more-->
<!-- CreateTime:2018/4/12 14:19:58 -->


<div id="toc"></div>

win10 可以很简单在我们的 app 使用自然输入，这篇文章主要翻译[https://blogs.windows.com/buildingapps/2015/09/08/going-beyond-keyboard-mouse-and-touch-with-natural-input-10-by-10/](https://blogs.windows.com/buildingapps/2015/09/08/going-beyond-keyboard-mouse-and-touch-with-natural-input-10-by-10/)  一些内容是参见[陈染大神](http://www.wangchenran.com/win10uwp%E5%BC%80%E5%8F%91-ink.html)

做法简单，我们有垃圾微软的`InkCanvas `，这个控件可以手写，需要我们在页面使用他：

```xml
<Grid>
  <InkCanvas x:Name="ink_canvas"/>
</Grid>
```

然后我们就可以写出我们的字，试试使用鼠标在程序写字。下面的不是我写的，是垃圾微软的。

![这里写图片描述](http://az648995.vo.msecnd.net/win/2015/09/1_ink.png)

`InkPresenter`可以获取 InkCanvas 基础对象，可以设置输入为笔，触摸，鼠标，上面那个是从微软拿来，因为我是在用电脑。

为了画出上面的图，我们可以设置`ink_canvas.InkPresenter.InputDeviceTypes= CoreInputDeviceTypes.Mouse;`如果我们有鼠标还要在手机运行，我们可以来`|`然后写手机，这样就可以使用多个方法。

```csharp
        public MainPage()
        {
            this.InitializeComponent();
            ink_canvas.InkPresenter.InputDeviceTypes= CoreInputDeviceTypes.Mouse;
        }
```

![这里写图片描述](http://img.blog.csdn.net/20160412164054442)

如果我们需要输入笔和鼠标 `ink_canvas.InkPresenter.InputDeviceTypes= CoreInputDeviceTypes.Mouse|CoreInputDeviceTypes.Pen;` ，关于这个枚举，参见[C＃枚举中使用Flags特性](http://lindexi.oschina.io/lindexi//post/C-%E6%9E%9A%E4%B8%BE%E4%B8%AD%E4%BD%BF%E7%94%A8Flags%E7%89%B9%E6%80%A7/)

画出的线我们也可以设置 线大小，颜色，请看代码

```csharp
            InkDrawingAttributes attribute = ink_canvas.InkPresenter.CopyDefaultDrawingAttributes();

            attribute.Color = Windows.UI.Colors.Crimson;//颜色
            attribute.PenTip = PenTipShape.Rectangle;//笔尖类型设置
            attribute.PenTipTransform = System.Numerics.Matrix3x2.CreateRotation((float)Math.PI / 4);////笔尖形状矩阵
            attribute.Size = new Size(2, 6);//画笔粗细

            ink_canvas.InkPresenter.UpdateDefaultDrawingAttributes(attribute);
```
## 保存，修改，加载ink

我们可以给用户选择他当前使用橡皮擦、铅笔还是他需要的。
我们给用户按钮铅笔，橡皮擦

```xml
    <Grid Background="{ThemeResource ApplicationPageBackgroundThemeBrush}">
        <Grid>
            <Grid.RowDefinitions>
                <RowDefinition />
                <RowDefinition Height="Auto"/>
            </Grid.RowDefinitions>
            <InkCanvas x:Name="ink_canvas" Grid.RowSpan="2" />
            <CommandBar Grid.Row="1">
                <AppBarButton Icon="Edit" Click="pencil"/>
                <AppBarButton Click="eraser">
                    <AppBarButton.Icon>
                        <BitmapIcon UriSource="ms-appx:///Assets/eraser_128px_1197233_easyicon.net.ico"/>
                    </AppBarButton.Icon>
                </AppBarButton>
            </CommandBar>
        </Grid>
    </Grid>
```
点击时，修改笔为橡皮擦或其他的，只需要设置当前的笔

```csharp
        private void eraser(object sender, RoutedEventArgs e)
        {
            ink_canvas.InkPresenter.InputProcessingConfiguration.Mode =
    InkInputProcessingMode.Erasing;
        }

        private void pencil(object sender, RoutedEventArgs e)
        {
            ink_canvas.InkPresenter.InputProcessingConfiguration.Mode =
                InkInputProcessingMode.Inking;
        }
```

点击橡皮可以擦掉，但是有些诡异，大家可以自己去写，自己去玩，就知道

接下来告诉大家，如何
保存墨迹

```csharp
            FileSavePicker picker = new FileSavePicker
            {
                SuggestedStartLocation = Windows.Storage.Pickers.PickerLocationId.PicturesLibrary
            };//建议我们选择在图片，其实这个不用写
            picker.FileTypeChoices.Add("Gif", new
            System.Collections.Generic.List<string> { ".gif" });//类型gif，其实是isf
            //名称
            picker.SuggestedFileName = "http://blog.csdn.net/lindexi_gd";
            StorageFile file = await picker.PickSaveFileAsync();
            if (null != file)
            {
                try
                {
                    using (IRandomAccessStream stream = await file.OpenAsync(FileAccessMode.ReadWrite))
                    {
                        await ink_canvas.InkPresenter.StrokeContainer.SaveAsync(stream);
                    }
                }
                catch (Exception ex)
                {
                    //http://blog.csdn.net/lindexi_gd
                }
            }
```

陈染大神的保存，我们上面保存是 gif

```csharp
 //声明一个流来存储墨迹信息
    IRandomAccessStream stream = new InMemoryRandomAccessStream();
    //保存墨迹信息到流
    //拿到流了就可以随意处置墨迹了，可以保持到App内部 也可以保存为文件，我们直接保存为文件
    await InkCanvas.InkPresenter.StrokeContainer.SaveAsync(stream);
    //创建一个文件保存对话框
    var picker = new Windows.Storage.Pickers.FileSavePicker
    {
        SuggestedStartLocation = Windows.Storage.Pickers.PickerLocationId.DocumentsLibrary
    };
    //文件类型
    picker.FileTypeChoices.Add("INK files", new List<string>() { ".ink" });
    //弹出保存对话框
    var file = await picker.PickSaveFileAsync();
    if (file == null) return;
 
    CachedFileManager.DeferUpdates(file);
    //将流转为byte
    var bt = await ConvertImagetoByte(stream);
    //写入文件
    await Windows.Storage.FileIO.WriteBytesAsync(file, bt);
    //保存
    await CachedFileManager.CompleteUpdatesAsync(file);

private async Task<byte[]> ConvertImagetoByte(IRandomAccessStream fileStream)
{
    //IRandomAccessStream fileStream = await image.OpenAsync(FileAccessMode.Read);
    var reader = new Windows.Storage.Streams.DataReader(fileStream.GetInputStreamAt(0));
    await reader.LoadAsync((uint)fileStream.Size);
 
    byte[] pixels = new byte[fileStream.Size];
 
    reader.ReadBytes(pixels);
 
    return pixels;
}
```

保存的东西可以加载，需要加载第一步是获得文件

```csharp
           //创建一个文件选择器
           var picker = new Windows.Storage.Pickers.FileOpenPicker
           {
               SuggestedStartLocation = Windows.Storage.Pickers.PickerLocationId.DocumentsLibrary
           };
           //规定文件类型
           picker.FileTypeFilter.Add(".ink");
           //显示选择器
           var pickedFile = await picker.PickSingleFileAsync();
           if (pickedFile != null)
           {
               var file = await pickedFile.OpenReadAsync();
               //加载墨迹
               await InkCanvas.InkPresenter.StrokeContainer.LoadAsync(file);
           }
```
如何获得文件参见：[win10 uwp 保存用户选择文件夹](http://lindexi.oschina.io/lindexi//post/win10-uwp-%E4%BF%9D%E5%AD%98%E7%94%A8%E6%88%B7%E9%80%89%E6%8B%A9%E6%96%87%E4%BB%B6%E5%A4%B9/)

## UWP 手写清理笔画

我们写完一个字需要清理我们笔画，可以使用clear

```csharp
ink.InkPresenter.StrokeContainer.Clear();
```

## 手写识别

```csharp
    //手写识别
    var container = new InkRecognizerContainer();
    //使用墨迹识别
    var result = await container.RecognizeAsync(InkCanvas.InkPresenter.StrokeContainer, InkRecognitionTarget.All);
    //获取识别结果  InkRecognitionResult 对象中还能获取候选字
    var txt = result[0].GetTextCandidates()[0];
```
手写识别来自 [http://www.wangchenran.com/win10uwp开发-ink.html](http://www.wangchenran.com/win10uwp开发-ink.html)

但是我们每次需要使用`InkCanvas`需要使用很多按钮，微软给了我们`Ink Toolbar `可以简单使用。
扩展下载：[https://visualstudiogallery.msdn.microsoft.com/58194dfe-df44-4c4e-893a-1eca40675269](https://marketplace.visualstudio.com/items?itemName=InkToolbarControlTeam.InkToolbarcontrolforUniversalWindowsapps)

![这里写图片描述](http://img.blog.csdn.net/20160901165925839) 

首先安装该工具扩展，然后引用InkToolbar Control.dll，接着在View中声明控件:
   

```csharp
xmlns:ink="using:Microsoft.Labs.InkToolbarControl"
 
 
<ink:InkToolbar x:Name="bar_InkTool"
 
 
TargetInkCanvas="{x:Bind InkCanvas}"
 
 
VerticalAlignment="Top" HorizontalAlignment="Right" />
```

`TargetInkCanvas`属性bind到要设置的`InkCanvas`上即可。

## 无法识别手写

首先我们手写需要安装。

如果我们没法识别，那么检查设置时间语言，检查安装语言，下载手写

那么我们可以使用

```csharp
            var container = new InkRecognizerContainer();
            foreach (var temp in container.GetRecognizers())
            {
                Text.Text += temp.Name + "\r\n";
            }
```

来看我们安装了哪些，有安装才能使用

源代码 https://github.com/lindexi/UWP/tree/master/uwp/src/Ink

## 语音

现在很多人都是使用语音输入，把文字转为语音我已经写了一篇博客。

我们需要先有麦克风，需要权限

![这里写图片描述](http://img.blog.csdn.net/20160416103452875)

首先我们需要设置语言，因为需要的识别，可以使用 web 的接口，所以就需要添加麦克风、网络的权限。

下面的代码就是告诉用户需要输入的内容，然后进行转换。

```csharp
            Language language = SpeechRecognizer.SystemSpeechLanguage;
            speechRecognizer = new SpeechRecognizer(language);

            // 使用web
            SpeechRecognitionTopicConstraint web_search_grammar = new SpeechRecognitionTopicConstraint(SpeechRecognitionScenario.WebSearch, "webSearch"); 
            speechRecognizer.Constraints.Add(web_search_grammar);

            speechRecognizer.UIOptions.AudiblePrompt = "你想要说什么";
            speechRecognizer.UIOptions.ExampleText = "http://blog.csdn.net/lindexi_gd";

            SpeechRecognitionCompilationResult compilation_result = await speechRecognizer.CompileConstraintsAsync();
            if (compilation_result.Status == SpeechRecognitionResultStatus.Success)
            {
                // 识别
                IAsyncOperation<SpeechRecognitionResult> recognition_operation = speechRecognizer.RecognizeWithUIAsync();
                SpeechRecognitionResult speech_recognition_result = await recognition_operation;
                SpeechRecognitionConfidence confidence = speech_recognition_result.Confidence;//置信度
                string text = speech_recognition_result.Text;//获取语音
            }
```

语音：https://msdn.microsoft.com/zh-cn/library/windows/apps/dn596121.aspx
 
http://stackoverflow.com/questions/32153880/how-to-render-inkcanvas-to-an-image-in-uwp-windows-10-application/32551620

https://blogs.windows.com/buildingapps/2015/09/08/going-beyond-keyboard-mouse-and-touch-with-natural-input-10-by-10/

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。