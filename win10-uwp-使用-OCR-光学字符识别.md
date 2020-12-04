
# win10 uwp 使用 OCR 光学字符识别

在 UWP 里面可以很方便通过 Windows.Media.Ocr.OcrEngine 识别图片的字符，其实老周有写过这一篇技术博客，今天有小伙伴在问如何实现，我还以为老周的博客过时了，于是重新复制老周的代码跑了一次，然后就通过了

<!--more-->


<!-- CreateTime:4/6/2020 12:36:21 PM -->



在老周的 [【Win10 应用开发】OCR识别](https://www.cnblogs.com/tcjiaan/p/4491906.html ) 博客还是 UAP 的代码，此时 UWP 还没发布，不过在 UWP 发布之后也没有改这部分的 API 也就是可以直接复制代码运行

欢迎小伙伴新建一个 UWP 应用，在某个按钮的点击事件里面复制下面的代码

```csharp
        private async void Button_OnClick(object sender, RoutedEventArgs e)
        {
            var picker = new Windows.Storage.Pickers.FileOpenPicker();
            picker.FileTypeFilter.Add(".png");
            // 选择文件
            var imgFile = await picker.PickSingleFileAsync();

            if(imgFile != null)
            {
                using (var inStream = await imgFile.OpenReadAsync())
                {
                    // 解码图片
                    var decoder = await Windows.Graphics.Imaging.BitmapDecoder.CreateAsync(Windows.Graphics.Imaging.BitmapDecoder.PngDecoderId, inStream);

                    // 获取图像
                    var swbmp = await decoder.GetSoftwareBitmapAsync();
                    // 准备识别
                    Windows.Globalization.Language lang = new Windows.Globalization.Language("zh-CN");
                    // 判断是否支持简体中文识别
                    if (Windows.Media.Ocr.OcrEngine.IsLanguageSupported(lang))
                    {
                        var engine = Windows.Media.Ocr.OcrEngine.TryCreateFromLanguage(lang);
                        if (engine != null)
                        {
                            var result = await engine.RecognizeAsync(swbmp);
                            if (result != null)
                            {
                                var dialog = new Windows.UI.Popups.MessageDialog($"识别内容 {result.Text}");
                                await dialog.ShowAsync();
                            }
                        }
                    }
                    else
                    {
                        var dialog = new Windows.UI.Popups.MessageDialog("不支持简体中文的识别。");
                        await dialog.ShowAsync();
                    }
                }
            }
        }
```

上面代码和老周的博客有一点不同的是我添加了很多命名空间，这样大概复制上面代码就可以跑起来了

代码的主要逻辑是 `var engine = Windows.Media.Ocr.OcrEngine.TryCreateFromLanguage(lang);` 创建识别引擎，然后通过 `var result = await engine.RecognizeAsync(swbmp);` 解析文本

更多细节还请看老周的 [【Win10 应用开发】OCR识别 - 东邪独孤 - 博客园](https://www.cnblogs.com/tcjiaan/p/4491906.html )

本文代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/1f21e3f9d8d906a40d922d9195cdccad1c1fe3af/CetursearhebirLefelembemki) 其实代码是一个新建的项目，也就加上一个按钮，在按钮里面添加上面代码





<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。