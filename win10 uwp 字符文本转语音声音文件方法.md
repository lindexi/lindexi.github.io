# win10 uwp 字符文本转语音声音文件方法

在 UWP 中，支持将传入的字符串文本内容转换为音频语音，可以将这个语音声音通过 MediaElement 播放，或者将这个音频保存到文件里面

<!--more-->
<!-- CreateTime:2020/10/28 10:39:38 -->



本文的方法是通过 SpeechSynthesizer 类提供的将 文本字符串 转换为 wav 的 Stream 对象实现的

核心的转换字符文本作为音频 Stream 代码如下

```csharp
using (SpeechSynthesizer synthesizer = new SpeechSynthesizer())
{
    SpeechSynthesisStream stream = await synthesizer.SynthesizeTextToStreamAsync(word);
}
```

上面代码的 word 就是传入的字符串文本，可以是一个单词也可以是一个句子或一段话

在 UWP 中使用如上面代码就可以用到 UWP 自带的语音合成技术的将文本转换为语音的功能

在拿到 SpeechSynthesisStream 之后，可以进行播放或者保存到文件

进行播放时需要使用到 MediaElement 控件，在 XAML 中先添加 MediaElement 控件，代码如下

```xml
        <MediaElement x:Name="MediaElement"></MediaElement>
```

在后台代码通过 SetSource 方法可以设置如上的音频对象进行博客

```csharp
     MediaElement.SetSource(stream, stream.ContentType);
     MediaElement.Play();
```

默认的 `stream.ContentType` 就是 wav 格式

而保存到音频数据到文件可以采用如下方法

```csharp
using (var wordFileStream = await wordFile.OpenStreamForWriteAsync())
{
    await stream.AsStreamForRead().CopyToAsync(wordFileStream);
}
```

在上面代码中的 wordFile 是一个 StorageFile 对象，可以通过用户选择等方式获取

本文代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/b5bc99a1/LinemlallledurKaicawkeedaykerewho ) 欢迎小伙伴访问

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。  
