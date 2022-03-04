# win10 uwp 选择文本转语音的机器人

在 UWP 里，可以非常方便将某个文本转换为音频语音，转换时，将会根据输入的内容以及本机所安装的语言库选择一位机器人帮忙将输入的文本转换为语音。本文来告诉大家如何切换文本转语音的机器人，例如从默认的女声转换为男声，如选择 Kangkang 或 Huihui 等特定机器人帮助转换语音

<!--more-->
<!-- CreateTime:2022/2/7 8:37:25 -->

<!-- 发布 -->

刚好从卢老师那里接了一个任务，录制 dotnet 的 20 周年的祝贺视频。然而过年生活太好的我嗓子沙哑了，于是本来普通话就说得不标准的我开始寻求起代码之神的帮助，好在翻到了自己的博客，找到了 [win10 uwp 字符文本转语音声音文件方法](https://blog.lindexi.com/post/win10-uwp-%E5%AD%97%E7%AC%A6%E6%96%87%E6%9C%AC%E8%BD%AC%E8%AF%AD%E9%9F%B3%E5%A3%B0%E9%9F%B3%E6%96%87%E4%BB%B6%E6%96%B9%E6%B3%95.html) 这篇博客，开始按照此方式录制，却发现了默认语音不是 Kangkang 机器人的。在 UWP 里的 SpeechSynthesizer 的 Voice 属性可以让咱设置所采用的机器人，但是此 VoiceInformation 对象却不能创建，不得不赞叹一下 API 设计者的强大

<!-- 你好，我是林德熙 , 我是 2006 年开始接触 dot net 的，我的第一个项目因为年代过于久远已忘记, 因为当年 dot net 是最强大无敌的原因，我选择了 dot net ，新一代的 dot net ， 我最喜欢它的全方位无所限制的开发功能，可快速开发顶层业务，也可接触最底层做极致的优化，希望 dot net 在中国有更加辉煌的发展! 祝贺 dot net 20周年生日快乐 -->

按照我也不知道哪学到的知识，文本转语音所采用的 TTS 需要依靠本机所安装的语言库，可以在注册表的 `HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Speech_OneCore\Voices\Tokens\` 路径下获取到当前本机所安装的语言库。也就是说即使自己开发了此功能，在用户端能否使用，完全需要取决于用户端是否安装了对应的语言包

在 UWP 下，不提供 VoiceInformation 对象的创建方法，必须通过 SpeechSynthesizer 的 AllVoices 属性获取本机已安装的机器人，从里面挑选一个用来设置。例如以下代码就是挑选 Kangkang 机器人用来帮忙转文本

```csharp
using Windows.Media.SpeechSynthesis;

            using (SpeechSynthesizer synthesizer = new SpeechSynthesizer())
            {
                VoiceInformation voice = SpeechSynthesizer.AllVoices.FirstOrDefault(v => v.Id.Contains("zhCN_KangkangM"));
                synthesizer.Voice = voice;
            }
```

以上的 AllVoices 属性是静态属性，获取时，即可拿到本机所有的已安装的机器人，可以自己遍历。但是由于这是一个 COM 对象，在 VS 调试使用相对不方便，推荐大家自己写一次循环获取一下

接下来的转换文本为语音的方法就之前博客的差不多，代码如下

```csharp
        private async void Button_OnClick(object sender, RoutedEventArgs e)
        {
            using (SpeechSynthesizer synthesizer = new SpeechSynthesizer())
            {
                VoiceInformation voice = SpeechSynthesizer.AllVoices.FirstOrDefault(v => v.Id.Contains("zhCN_KangkangM"));
                synthesizer.Voice = voice;
                var text = InputTextBox.Text;
                try
                {
                    SpeechSynthesisStream stream = await synthesizer.SynthesizeTextToStreamAsync(text);
                    using (stream)
                    {
                        FileSavePicker savePicker = new FileSavePicker();
                        savePicker.FileTypeChoices.Add("音频", new[] { ".wav" });

                        var result = await savePicker.PickSaveFileAsync();
                        var wordFile = result;

                        using (var wordFileStream = await wordFile.OpenStreamForWriteAsync())
                        {
                            await stream.AsStreamForRead().CopyToAsync(wordFileStream);
                        }
                    }
                }
                catch (Exception exception)
                {
                    Debug.WriteLine(exception);
                }
            }
        }
```

界面代码如下

```xaml
    <Grid>
        <TextBox x:Name="InputTextBox" Margin="10,10,10,100" HorizontalTextAlignment="Left" HorizontalAlignment="Left" TextWrapping="Wrap"/>
        <Button Margin="10,10,10,10" VerticalAlignment="Bottom" Content="文字转语音" Click="Button_OnClick"></Button>
    </Grid>
```

本文代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/bb7b63b26d352f6ead931e00b19ed6d32e57e735/LinemlallledurKaicawkeedaykerewho ) 欢迎小伙伴访问

更多请看 [SpeechSynthesizer 类 (System.Speech.Synthesis) Microsoft Docs](https://docs.microsoft.com/zh-cn/dotnet/api/system.speech.synthesis.speechsynthesizer?view=netframework-4.8)

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
