---
title: dotnet 基于 FFmpeg 实现图片加多音频批量合成视频方法
description: 本文将告诉大家如何使用 C# 封装 FFmpeg 实现批量将单张图片与多个音频片段合成视频段落，再拼接为完整视频的方案，支持自定义编码参数、全异步执行，可轻松集成到各类.NET 应用中。
tags: dotnet
category: 
---

<!-- CreateTime:2026/05/09 07:07:29 -->

<!-- 发布 -->
<!-- 博客 -->

本文内容由人类主导 AI 辅助编写

我最近需要做一个批量生成讲解类视频的功能，每个视频由多个页面片段组成，每个片段对应一张 PPT 图片和多段讲解音频，需要将每个片段的图片和音频合成为一段视频，再按顺序拼接成完整的视频。选择 FFmpeg 作为底层工具是因为它跨平台、编码能力强、性能优异，完全满足批量生成的需求。

整体实现思路分成两步：第一步是逐个处理每个视频片段，将单张图片与该片段的所有音频拼接后合成单段视频；第二步是将所有生成的单段视频无损拼接为最终的完整视频。这种分步的方式不仅方便排错，而且拼接阶段直接复制流不需要重新编码，速度极快。

## 核心 FFmpeg 参数详解

整个方案的核心是 FFmpeg 命令的参数设计，下面分别拆解单片段合成和多片段拼接的参数含义。

### 单片段合成参数

单片段合成的作用是将一张静态图片和多段音频合并为一段和音频时长一致的视频，对应的参数如下：

```
-loop 1 -r {帧率} -i {图片路径} -f concat -safe 0 -i {音频列表文件路径} -vf "scale={宽度}:{高度}:force_original_aspect_ratio=decrease,pad={宽度}:{高度}:(ow-iw)/2:(oh-ih)/2:black" -c:v libx264 -b:v {视频码率} -c:a aac -b:a {音频码率} -pix_fmt yuv420p -shortest -y {输出路径}
```

逐段解释各个参数的作用：

1. `-loop 1 -r {帧率} -i {图片路径}`：`-loop 1` 表示循环读取输入的静态图片，这样才能生成和音频时长匹配的连续视频；`-r` 指定输出视频的帧率，比如设置为 24 的话图片就会每秒重复 24 帧。
2. `-f concat -safe 0 -i {音频列表文件路径}`：使用 FFmpeg 的 concat 分离器直接拼接多个音频文件，不需要重新编码，速度极快；`-safe 0` 允许读取绝对路径的文件，避免因为路径格式问题报错。音频列表文件的格式为每行一条 `file '音频路径'`，代码中已经内置了路径的单引号转义逻辑，避免特殊字符导致解析失败。
3. `-vf` 后面跟着的是视频滤镜参数，用来统一输出视频的分辨率：首先 `scale` 滤镜会按照原图片的宽高比缩放到不超过目标分辨率的大小，`force_original_aspect_ratio=decrease` 表示只缩小不放大，避免小图拉伸导致模糊；然后 `pad` 滤镜会把缩放后的图片居中填充到目标分辨率，上下左右不够的地方用黑色填充，保证所有分段的分辨率完全一致。
4. `-c:v libx264 -b:v {视频码率} -c:a aac -b:a {音频码率} -pix_fmt yuv420p`：编码参数，视频使用 H.264 编码，音频使用 AAC 编码，同时支持自定义码率；`pix_fmt yuv420p` 是为了兼容绝大多数播放器，避免生成的视频在部分移动设备上无法播放。
5. `-shortest -y {输出路径}`：`-shortest` 表示当最短的输入流结束时就停止编码，也就是视频时长和拼接后的音频时长自动保持一致；`-y` 表示直接覆盖已存在的输出文件，不需要额外的交互确认。

### 多片段拼接参数

所有分段视频生成完成后，就可以执行拼接操作，对应的参数非常简单：

```
-f concat -safe 0 -i {视频列表文件路径} -c copy -y {输出路径}
```

这里最核心的参数是 `-c copy`，表示直接复制视频和音频流，不需要重新编码，所以拼接速度非常快，而且完全没有画质损失，适合大批量视频的拼接场景。

## C# 封装实现细节

我将整个合成逻辑封装为 `FFmpegVideoComposer` 类，上层使用非常简单，不需要关心 FFmpeg 命令的构造细节。

首先是构造函数的设计，初始化时会做前置校验，确保 FFmpeg 文件存在，同时自动生成临时工作目录用来存储中间文件：

```csharp
/// <summary>
/// 构造函数
/// </summary>
/// <param name="ffmpegExe">FFmpeg可执行文件路径</param>
/// <param name="encodeSettings">视频编码配置，不传则用默认值</param>
/// <param name="workingDirectory">工作目录，不传则使用系统临时目录</param>
/// <param name="logHandler">日志回调，上层可自定义日志输出方式</param>
public FFmpegVideoComposer(
    FileInfo ffmpegExe,
    VideoEncodeSettings? encodeSettings = null,
    DirectoryInfo? workingDirectory = null,
    LogHandler? logHandler = null)
```

核心合成方法 `ComposeAsync` 全程异步执行，支持取消令牌，随时可以终止任务：

```csharp
/// <summary>
/// 合成视频核心方法
/// </summary>
/// <param name="segments">按播放顺序排列的视频分段列表</param>
/// <param name="outputFile">最终输出视频文件</param>
/// <param name="cancellationToken">取消令牌</param>
/// <returns>是否合成成功</returns>
public async Task<bool> ComposeAsync(
    IReadOnlyList<VideoSegment> segments,
    FileInfo outputFile,
    CancellationToken cancellationToken = default)
```

执行 FFmpeg 命令的部分做了专门的优化，使用异步方式读取进程输出，避免缓冲区满导致的死锁问题；如果任务被取消，会自动杀死 FFmpeg 进程，避免残留后台进程占用资源。同时 `FFmpegVideoComposer` 实现了 `IAsyncDisposable` 接口，退出作用域时会自动清理所有临时文件，不需要上层手动处理资源释放。

具体实现代码，可到本文末尾获取拉取代码的方法，拉取代码了解细节实现

## 使用示例

使用方式非常简单，只需要准备好视频分段列表，初始化合成器之后调用合成方法即可：

```csharp
// 准备视频分段列表，每个分段对应一张图片和多段音频
var segments = new List<VideoSegment>();
for (int i = 0; i < 5; i++)
{
    segments.Add(new VideoSegment()
    {
        ImageFile = new FileInfo($"slide_{i:D4}.png"),
        AudioFiles = new List<FileInfo>()
        {
            new FileInfo($"slide_{i:D4}/audio_0000.mp3"),
            new FileInfo($"slide_{i:D4}/audio_0001.mp3")
        }
    });
}

// 初始化合成器，使用 await using 会自动清理临时资源
await using var composer = new FFmpegVideoComposer(
    ffmpegExe: new FileInfo("ffmpeg.exe"), // 替换为你的 FFmpeg 可执行文件路径
    encodeSettings: new VideoEncodeSettings()
    {
        Width = 1920,
        Height = 1080,
        Fps = 24,
        VideoBitrate = "2M",
        AudioBitrate = "128k"
    },
    logHandler: (level, message) => Console.WriteLine($"[{level}] {message}"));

// 执行合成
var outputFile = new FileInfo("output.mp4");
bool success = await composer.ComposeAsync(segments, outputFile);
if (success)
{
    Console.WriteLine($"视频合成成功，路径：{outputFile.FullName}");
}
```

## 注意事项

1. 所有输入的图片和音频文件需要提前确认存在，代码内置了前置校验，文件不存在时会提前返回错误信息，避免执行到一半失败。
2. 建议使用最新的稳定版 FFmpeg，避免因为版本兼容性问题导致编码失败。
3. 如果需要合成大量视频，可以复用同一个 `FFmpegVideoComposer` 实例，减少临时目录创建销毁的开销。

以上方案已经在多个项目中验证过，生成的视频兼容性好，性能也足够应对批量生成的需求，大家可以根据自己的业务场景扩展更多功能，比如添加转场特效、字幕等。

## 代码

本文代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/e4eedfcc26ac40d7ad1ddea0f56a1f34658db15d/SemanticKernelSamples/VideoComposer) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/blob/e4eedfcc26ac40d7ad1ddea0f56a1f34658db15d/SemanticKernelSamples/VideoComposer) 上，可以使用如下命令行拉取代码。我整个代码仓库比较庞大，使用以下命令行可以进行部分拉取，拉取速度比较快

先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin e4eedfcc26ac40d7ad1ddea0f56a1f34658db15d
```

以上使用的是国内的 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码，将 gitee 源换成 github 源进行拉取代码。如果依然拉取不到代码，可以发邮件向我要代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin e4eedfcc26ac40d7ad1ddea0f56a1f34658db15d
```

获取代码之后，进入 SemanticKernelSamples/VideoComposer 文件夹，即可获取到源代码

更多技术博客，请参阅 [博客导航](https://blog.lindexi.com/post/%E5%8D%9A%E5%AE%A2%E5%AF%BC%E8%88%AA.html )
