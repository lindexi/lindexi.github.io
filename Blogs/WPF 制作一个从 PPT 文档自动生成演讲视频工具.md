---
title: WPF 制作一个从 PPT 文档自动生成演讲视频工具
description: 本文将告诉大家如何基于多模态大模型、语音合成能力和 FFMpeg 实现自动将 PPT 文档转换为带专业讲解的演讲视频，项目代码拉取后即可直接运行。我现在给的提示词炼丹效果是针对于教育垂直领域，整个代码完全开源，欢迎二次创作
tags: WPF
category: 
---

<!-- CreateTime:2026/05/09 07:08:02 -->

<!-- 发布 -->
<!-- 博客 -->

本文内容由人类主导 AI 辅助编写

平时做技术分享或者课程录制的时候，经常需要对着 PPT 录制讲解视频，每次重录都要花费大量时间。于是我做了这个小工具，只需要导入 PPT 文件，就能自动生成连贯的演讲视频，全程不需要人工干预，效果如下：

效果演示视频： <http://cdn.lindexi.site/courseware_speech_20260507211402.mp4>

软件运行界面：

<!-- ![](image/WPF 制作一个从 PPT 文档自动生成演讲视频工具/WPF 制作一个从 PPT 文档自动生成演讲视频工具0.gif) -->
![](https://img2024.cnblogs.com/blog/1080237/202605/1080237-20260509071039103-217114824.gif)

## 实现原理

整个工具的核心逻辑分为三步：

1. 多模态大模型处理：提取 PPT 每一页的文本内容和页面截图，同时传入整份 PPT 的上下文信息，让大模型生成每一页连贯的讲解脚本，保证前后内容衔接自然不重复。
2. 语音合成：将大模型生成的每一页讲稿，调用语音合成服务转换为自然流畅的音频。
3. 视频合成：使用 FFMpeg 将 PPT 页面截图和对应的音频按顺序拼接，最终生成完整的 MP4 视频文件。

## 快速开始

本文代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/7ff72da8af31ddf01ba28ada7ee9adeef4abcc13/WPFDemo/JawjeleceeYairlubelhearrene) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/7ff72da8af31ddf01ba28ada7ee9adeef4abcc13/WPFDemo/JawjeleceeYairlubelhearrene) 上，可以使用如下命令行拉取代码。我整个代码仓库比较庞大，使用以下命令行可以进行部分拉取，拉取速度比较快

先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin 7ff72da8af31ddf01ba28ada7ee9adeef4abcc13
```

以上使用的是国内的 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码，将 gitee 源换成 github 源进行拉取代码。如果依然拉取不到代码，可以发邮件向我要代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin 7ff72da8af31ddf01ba28ada7ee9adeef4abcc13
```

拉取代码之后，进入 `WPFDemo/JawjeleceeYairlubelhearrene` 文件夹，打开 sln 解决方案即可直接编译运行。运行前需要提前准备好以下信息：

- 兼容 OpenAI API 格式的多模态大模型服务（比如豆包 ARK 平台的多模态模型），需要配置对应的 API Key、服务端点和模型 ID。
- 火山引擎 OpenSpeech 语音合成服务的 API Key 和对应资源 ID，用于生成讲稿音频。
- 本地准备好 FFMpeg 可执行文件，配置到工具的对应路径即可。
  - 可从 <https://www.ffmpeg.org/download.html> 官网下载 FFMpeg 可执行文件

自己注册账号是有免费额度的，实在不想注册的话，再私聊找我借账号吧

## 代码实现细节

整个工具采用 MVVM 架构，核心逻辑封装在 `CoursewareSpeechVideoGenerator` 服务类中，下面拆分各个模块的实现细节。

### PPT 内容提取

首先需要使用 OpenXML 解析 PPT 文件，提取每一页的文本内容，同时生成每一页的缩略图截图，封装到 `CoursewareMaterialInfo` 模型中，包含整个 PPT 的所有页面信息，代码如下：


```csharp
public class CoursewareMaterialInfo
{
    /// <summary>
    /// PPT所有页面的材料信息
    /// </summary>
    public List<CoursewareSlideMaterialInfo> SlideMaterialInfoList { get; set; } = [];
}

public class CoursewareSlideMaterialInfo
{
    /// <summary>
    /// 当前页面的文本内容
    /// </summary>
    public string ContentText { get; set; } = string.Empty;
    /// <summary>
    /// 当前页面的截图文件路径
    /// </summary>
    public string SlideThumbnailFilePath { get; set; } = string.Empty;
}
```

这里使用 OpenXML 提取文本，然后使用 COM 方式调用本机的 Office 组件进行 PPT 页面截图。细节逻辑还请自行阅读代码

### 讲稿生成逻辑

讲稿生成是整个工具的核心，为了保证生成的讲稿上下文连贯，每次调用大模型的时候，不仅传入当前页的文本和截图，还会传入整份 PPT 的全部文本和前面已经生成的所有讲稿内容，让大模型能够理解上下文，避免重复讲解，代码如下：

```csharp
var openAiClient = new OpenAIClient(new ApiKeyCredential(options.OpenAiApiKey), new OpenAIClientOptions
{
    Endpoint = options.OpenAiEndpoint,
    NetworkTimeout = TimeSpan.FromHours(1)
});
var chatClient = openAiClient.GetChatClient(options.OpenAiModel).AsIChatClient();

// 逐页生成讲稿
for (var i = 0; i < coursewareMaterialInfo.SlideMaterialInfoList.Count; i++)
{
    var slideMaterialInfo = coursewareMaterialInfo.SlideMaterialInfoList[i];
    var slideIndex = i + 1;
    
    var plainScriptText = await GenerateSlideScriptAsync(
        chatClient,
        allCoursewareText, // 整份PPT的全部文本
        slideIndex,
        slideMaterialInfo.ContentText, // 当前页文本
        previousScriptsBuilder.ToString(), // 前面已经生成的讲稿
        slideMaterialInfo.SlideThumbnailFilePath, // 当前页截图
        cancellationToken);
    
    slideInfoList.Add(new CoursewareSpeechSlideInfo(plainScriptText, slideMaterialInfo));
    previousScriptsBuilder.AppendLine(plainScriptText);
}
```

这里用了 `Microsoft.Extensions.AI` 抽象的 `IChatClient` 接口，所以只要是兼容 OpenAI API 格式的多模态模型都可以直接对接，不需要修改核心逻辑，比如对接豆包 ARK 服务的时候只需要替换对应的 Endpoint 和模型 ID 即可。为什么我这里没有选用 DeepSeek 呢？这是因为现在（2026年5月）的 DeppSeek 还没有多模态支持

具体的生成讲稿 GenerateSlideScriptAsync 逻辑核心就是拼接文本信息，以及将截图给到多模态的大模型，要求它输出讲稿内容。这部分涉及到了提示词炼丹，提示词内容比较长，我就不放在博客里面，大家可以拉取我的代码，了解细节内容

生成的讲稿内容示例如下：

<!-- ![](image/WPF 制作一个从 PPT 文档自动生成演讲视频工具/WPF 制作一个从 PPT 文档自动生成演讲视频工具0.png) -->
![](https://img2024.cnblogs.com/blog/1080237/202605/1080237-20260509071039945-392981072.png)

```
[上下文: 语气亲切活泼，带有课堂互动感，符合小学语文教师授课风格]
现在我们正式进入第1课时的学习，首先开启的是“激发兴趣，导入新课”环节。[停顿: 0.8秒]
大家看页面中间的“儿歌欣赏”字样，旁边还有一个小喇叭的图标，我们先来听一首和今天学习主题相关的趣味儿歌哦。[停顿: 1秒]
听完儿歌老师要考考大家，这是一首关于什么的儿歌呀？没错，答案就是我们今天的主题腊八粥，你看页面上蓝色标注的答案，你答对了吗？[停顿: 1秒]
接下来老师再问大家第二个问题，腊八粥是哪个节日会吃的传统美食呢？[停顿: 1.2秒]
太棒了，大家都记得，就是腊八节，每年农历腊月初八腊八节这天，喝一碗热乎香甜的腊八粥，是很多地方流传已久的习俗哦。[停顿: 0.8秒]
不知道大家还记不记得之前学过的老舍先生的《北京的春节》？那篇课文里是怎样介绍腊八粥的呢？[停顿: 1秒]
对哦，在《北京的春节》的第1自然段里，老舍先生就专门介绍了腊八粥用到的丰富食材，写得特别让人嘴馋。[停顿: 0.8秒]
除了我们刚刚提到的这些内容，你对腊八粥还有更多的了解吗？大家可以先在心里想一想，接下来我们就一起走近作家沈从文，看看他笔下的腊八粥又有着怎样的动人故事。
```

以上这些 `[]` 包括起来的内容，将在下一步语音合成逻辑进行处理

### 语音合成逻辑

讲稿生成完成之后，调用火山引擎的 OpenSpeech 服务逐页生成音频，这里封装了统一的合成配置，可以自定义发音人、语速、语调等参数：

```csharp
var authentication = OpenSpeechAuthentication.CreateWithApiKey(options.OpenSpeechApiKey, options.ResourceId);
var synthesisOptions = new CoursewareSpeechSynthesisOptions(authentication, options.Speaker);

// 逐页合成音频
foreach (var slideInfo in speechInfo.SlideInfoList)
{
    var audioFile = await OpenSpeechSynthesizer.SynthesizeAsync(
        slideInfo.ScriptText, 
        synthesisOptions, 
        cancellationToken);
    slideInfo.AudioFilePath = audioFile.FullName;
}
```

火山引擎的 OpenSpeech 提供了多种专业的发音人可选，生成的音频自然流畅，几乎听不出来是合成的语音。在合成语音的时候，需要先执行解析，提取出来脚本内容，比如上下文和停顿信息，这部分细节都在 CoursewareSpeechVideoGenerator.cs 文件里面，核心提取逻辑都是采用正则匹配，如果大家感兴趣，还请拉取代码了解细节

### 视频合成逻辑

最后一步就是使用 FFMpeg 将每一页的截图和对应的音频拼接成完整的视频，这里我封装了 `FFmpegVideoComposer` 类处理视频合成的逻辑，自动处理图片时长和音频的对齐：

```csharp
await using var ffmpegVideoComposer = new FFmpegVideoComposer(
    options.FfmpegExecutableFile,
    workingDirectory: options.OutputDirectory,
    logHandler: (level, message) => progress?.Report($"[{level}] {message}"));

// 逐页添加视频片段
foreach (var slideInfo in speechInfo.SlideInfoList)
{
    ffmpegVideoComposer.AddImageClip(
        slideInfo.SlideMaterialInfo.SlideThumbnailFilePath,
        slideInfo.AudioFilePath);
}

// 合成最终视频
var outputVideoFile = new FileInfo(Path.Combine(options.OutputDirectory.FullName, $"courseware_speech_{DateTime.Now:yyyyMMddHHmmss}.mp4"));
await ffmpegVideoComposer.ComposeAsync(outputVideoFile, cancellationToken);
```

合成的视频默认保存在输出目录下，文件名带时间戳，中间生成的临时音频和图片文件会在合成完成后自动清理，不会占用额外磁盘空间。

详细的技术细节请参阅： [dotnet 基于 FFmpeg 实现图片加多音频批量合成视频方法](https://blog.lindexi.com/post/dotnet-%E5%9F%BA%E4%BA%8E-FFmpeg-%E5%AE%9E%E7%8E%B0%E5%9B%BE%E7%89%87%E5%8A%A0%E5%A4%9A%E9%9F%B3%E9%A2%91%E6%89%B9%E9%87%8F%E5%90%88%E6%88%90%E8%A7%86%E9%A2%91%E6%96%B9%E6%B3%95.html )
<!-- [dotnet 基于 FFmpeg 实现图片加多音频批量合成视频方法 - lindexi - 博客园](https://www.cnblogs.com/lindexi/p/19998655 ) -->

## 注意事项

1. 建议使用支持长上下文的多模态模型，如果 PPT 页数很多，短上下文模型可能会因为上下文溢出导致生成的讲稿不连贯。
2. FFMpeg 建议使用较高版本，避免出现编码兼容问题。
3. 可以根据自己的需求修改大模型的提示词，自定义讲稿的风格，比如更口语化、更正式、更活泼等。
4. 工具已经实现了进度上报功能，生成过程中可以实时看到当前处理到第几页，剩余时间等信息。

提示词部分是我采用 [HyperAgent 思想的实践 制作 PPT 文档内容分析提示词](https://blog.lindexi.com/post/HyperAgent-%E6%80%9D%E6%83%B3%E7%9A%84%E5%AE%9E%E8%B7%B5-%E5%88%B6%E4%BD%9C-PPT-%E6%96%87%E6%A1%A3%E5%86%85%E5%AE%B9%E5%88%86%E6%9E%90%E6%8F%90%E7%A4%BA%E8%AF%8D.html ) 的方式炼丹获取的，大家可以换成其他提示词
<!-- [HyperAgent 思想的实践 制作 PPT 文档内容分析提示词 - lindexi - 博客园](https://www.cnblogs.com/lindexi/p/19919368 ) -->

## 相关博客

[dotnet Microsoft Agent Framework 配置调用工具后退出对话](https://blog.lindexi.com/post/dotnet-Microsoft-Agent-Framework-%E9%85%8D%E7%BD%AE%E8%B0%83%E7%94%A8%E5%B7%A5%E5%85%B7%E5%90%8E%E9%80%80%E5%87%BA%E5%AF%B9%E8%AF%9D.html )
<!-- [dotnet Microsoft Agent Framework 配置调用工具后退出对话 - lindexi - 博客园](https://www.cnblogs.com/lindexi/p/19773412 ) -->

[Microsoft Agent Framework 与 DeepSeek 对接](https://blog.lindexi.com/post/Microsoft-Agent-Framework-%E4%B8%8E-DeepSeek-%E5%AF%B9%E6%8E%A5.html )
<!-- [Microsoft Agent Framework 与 DeepSeek 对接 - lindexi - 博客园](https://www.cnblogs.com/lindexi/p/19413475 ) -->

[Office 使用 OpenXML SDK 解析文档博客目录](https://blog.lindexi.com/post/Office-%E4%BD%BF%E7%94%A8-OpenXML-SDK-%E8%A7%A3%E6%9E%90%E6%96%87%E6%A1%A3%E5%8D%9A%E5%AE%A2%E7%9B%AE%E5%BD%95.html )

更多技术博客，请参阅 [博客导航](https://blog.lindexi.com/post/%E5%8D%9A%E5%AE%A2%E5%AF%BC%E8%88%AA.html )
