# win10 uwp 使用油墨输入

win10可以很简单在我们的app使用自然输入，这篇文章主要翻译[https://blogs.windows.com/buildingapps/2015/09/08/going-beyond-keyboard-mouse-and-touch-with-natural-input-10-by-10/](https://blogs.windows.com/buildingapps/2015/09/08/going-beyond-keyboard-mouse-and-touch-with-natural-input-10-by-10/)  一些内容是参见[陈染大神](http://www.wangchenran.com/win10uwp%E5%BC%80%E5%8F%91-ink.html)

现在很多人还是使用笔和纸来记录，那么可以在电脑输入方式和之前使用的方式一样，很多用户觉得会方便。在win10 我们有一个简单的方法去让用户输入，`InkCanvas`。现在edge，OneNote这些都有使用`InkCanvas`。

我们可以在我们的页面

```
<Grid>
  <InkCanvas x:Name="ink_canvas"/>
</Grid>
```
![这里写图片描述](http://az648995.vo.msecnd.net/win/2015/09/1_ink.png)

`InkPresenter`可以获取InkCanvas基础对象，可以设置输入为笔，触摸，鼠标，上面那个是从微软拿来，因为我是在用电脑。

为了画出上面的图，我们可以设置`ink_canvas.InkPresenter.InputDeviceTypes= CoreInputDeviceTypes.Mouse;`

```
        public MainPage()
        {
            this.InitializeComponent();
            ink_canvas.InkPresenter.InputDeviceTypes= CoreInputDeviceTypes.Mouse;
        }
```

![这里写图片描述](http://img.blog.csdn.net/20160412164054442)

如果我们需要输入笔和鼠标`ink_canvas.InkPresenter.InputDeviceTypes= CoreInputDeviceTypes.Mouse|CoreInputDeviceTypes.Pen;`

画出的线我们也可以设置

```
            InkDrawingAttributes attribute = ink_canvas.InkPresenter.CopyDefaultDrawingAttributes();

            attribute.Color = Windows.UI.Colors.Crimson;//颜色
            attribute.PenTip = PenTipShape.Rectangle;//笔尖类型设置
            attribute.PenTipTransform = System.Numerics.Matrix3x2.CreateRotation((float)Math.PI / 4);////笔尖形状矩阵
            attribute.Size = new Size(2, 6);//画笔粗细

            ink_canvas.InkPresenter.UpdateDefaultDrawingAttributes(attribute);
```
##保存，修改，加载ink

我们可以给用户选择他当前使用橡皮擦、铅笔还是他需要的。
我们给用户按钮铅笔，橡皮擦

```
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

```
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
点击橡皮可以擦掉，但是有些诡异，大家可以自己去写
保存墨迹

```
            FileSavePicker picker = new FileSavePicker
            {
                SuggestedStartLocation = Windows.Storage.Pickers.PickerLocationId.PicturesLibrary
            };
            picker.FileTypeChoices.Add("Gif", new
            System.Collections.Generic.List<string> { ".gif" });
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
陈染大神的保存

```
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

保存的东西可以加载

```
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
##手写识别

```
    //手写识别
    var container = new InkRecognizerContainer();
    //使用墨迹识别
    var result = await container.RecognizeAsync(InkCanvas.InkPresenter.StrokeContainer, InkRecognitionTarget.All);
    //获取识别结果  InkRecognitionResult 对象中还能获取候选字
    var txt = result[0].GetTextCandidates()[0];
```
手写识别来自http://www.wangchenran.com/win10uwp开发-ink.html

但是我们每次需要使用`InkCanvas`需要使用很多按钮，微软给了我们`Ink Toolbar `可以简单使用。
   首先安装该工具扩展，然后引用InkToolbar Control.dll，接着在View中声明控件:
   

```
xmlns:ink="using:Microsoft.Labs.InkToolbarControl"
 
 
<ink:InkToolbar x:Name="bar_InkTool"
 
 
TargetInkCanvas="{x:Bind InkCanvas}"
 
 
VerticalAlignment="Top" HorizontalAlignment="Right" />
```
`TargetInkCanvas`属性bind到要设置的`InkCanvas`上即可。

##语音
现在很多人都是使用语音输入，把文字转为语音我已经写了一篇博客。
我们需要先有麦克风
![这里写图片描述](http://img.blog.csdn.net/20160416103452875)
首先我们需要设置语言
需要的识别，可以使用web
告诉用户需要输入
```
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