---
title: HyperAgent 思想的实践 制作 PPT 文档内容分析提示词
description: 本文记录我基于 HyperAgent 多代理协作思想，实现自动化分析 PPT 每一页内容、判断页面上下文作用，还能自动迭代优化提示词的方案，所有代码可直接复用，适配豆包、通义千问、本地 Ollama 多模态模型等多个接口。
tags: 
category: 
---

<!-- 发布 -->
<!-- 博客 -->

本文内容由 AI 辅助编写

这是我炼出来的提示词内容：

```
你是专业的PPT页面分析子代理，需严格基于给定的所有材料完成分析，禁止任何形式的编造、臆测。
你可获取的分析材料如下：

1. 整份PPT的全部文本：$(AllPptText)
2. 当前分析的页面序号：$(SlideIndex)
3. 当前页面的提取文本：$(CurrentPageText)
4. 此前所有页面的分析结果：$(PreviousResults)
5. 当前页面的完整截图

## 核心规则

1. 如实描述优先：所有内容必须完全来自当前页文本和截图，未明确标注的信息、不存在的内容绝对不能提及，不得对用途不明的元素主观脑补其作用；
2. 模块严格区分：两个输出模块边界清晰，【这页面包含了啥】仅做客观事实还原，不得加入任何作用、意义类的主观判断；【在页面上下文的作用】仅做逻辑关联分析，不得重复描述页面已有的元素细节；
3. 上下文分析必须锚定整份PPT：所有作用判断必须结合$(AllPptText)的整体结构和$(PreviousResults)的前后承接关系，逻辑必须符合PPT的实际内容排布，不得编造关联。

## 输出要求（严格按照以下两个维度结构化输出，不得增减模块）

### 1. 这页面包含了啥

客观罗列当前页所有实际存在的元素，包含但不限于：
- 各级标题、正文内容、知识点条目、标注的教材页码、特殊要求（如“背诵”）、高亮/下划线等格式属性；
- 所有图片、图表、插画、设计风格、背景元素；
- 引导问题、留白区域、空白文本框等元素；
- 与页面核心主题无关的冗余内容、突兀内容，需明确标注「未说明该内容与当前页面核心主题的关联」；
- 用途未明确的元素，需明确标注「用途未明确」。

### 2. 在页面上下文的作用

结合整份PPT的整体结构、前后页的内容承接关系，精准说明当前页的定位，禁止使用同质化套话，需明确包含以下信息：

- 该页在整个PPT的叙事/教学/结构逻辑中所属的模块（如单元目录页、单课导入页、知识点引入页、知识点总结页、自学引导页等）；
- 该页承接了前面哪些已讲内容/提出的问题/设定的框架；
- 该页为后续哪些内容做了铺垫/引出了什么新的知识点模块；
- 若该页存在前后呼应的内容（如解答前面提出的问题、呼应前面给出的框架），需明确说明对应关系。
```

## 背景

我日常工作中经常需要批量处理几十上百页的 PPT 文档，手动整理每一页的内容、梳理整个文档的逻辑结构耗时耗力，于是就想着用多代理协作的方式完成自动化分析，不需要手动反复调整提示词，让代理自己迭代优化分析效果。

## 代理角色设计

我设计了三个分工明确的 Agent 角色，相互配合完成整个流程：

- MainAgent: 负责撰写 SubAgent 的提示词，根据中立 Agent 的评价和 SubAgent 的运行结果，迭代优化 SubAgent 的提示词
- 中立 Agent: 客观评价 SubAgent 的分析结果，指出事实错误、逻辑问题、幻觉内容等
- SubAgent：输入是 MainAgent 生成的提示词、整份 PPT 的所有文本、当前页的文本和截图、之前页面的分析结果，输出当前页面的客观内容描述，以及该页面在整个 PPT 上下文里的作用

## 前置依赖

整个实现依赖以下几个组件：

1. Microsoft.Office.Interop.PowerPoint：用于导出 PPT 每一页的高清截图，需要注意 COM 对象的释放，避免 PPT 进程残留
2. DocumentFormat.OpenXml 3.5.1 版本：用于读取 PPTX 格式文档里的所有文本内容，包括标题、正文、表格、备注等
3. Microsoft Agent Framework：用于实现多代理的调度、工具调用、流式输出处理
4. 豆包 ARK 接口 / 本地 Ollama 多模态模型：负责大模型的推理能力



## PPT 内容和截图获取

首先需要实现从 PPT 里提取文本和导出截图的能力，我封装了两个核心类：

第一个是 `PowerPresentationProvider` 类，用于导出每一页的高清截图，支持 STA 线程调用，自动处理 COM 释放和临时文件清理，代码如下：

```csharp
internal sealed class PowerPresentationProvider : IDisposable
{
    private const string PasswordMark = "::PASSWORD::";
    private const double ExportDpi = 192;
    private FileInfo? _tempPresentationFile;
    private bool _disposedValue;

    public Application? Application { get; private set; }

    public Presentation? Presentation { get; private set; }

    /// <summary>
    /// 默认的打开超时时间，默认是 <see cref="Timeout.InfiniteTimeSpan"/> 的值
    /// </summary>
    public static TimeSpan DefaultOpenPowerPresentationTimeout { set; get; } = Timeout.InfiniteTimeSpan;

    /// <summary>
    /// 打开文件
    /// </summary>
    /// <param name="filePath">PPT文件地址</param>
    /// <param name="shouldCopyFile">是否需要将原文件复制到临时文件</param>
    public void OpenPowerPresentation(FileInfo filePath, bool shouldCopyFile = true)
    {
        ArgumentNullException.ThrowIfNull(filePath);
        if (!filePath.Exists)
        {
            throw new FileNotFoundException("未找到指定的 PowerPoint 文件。", filePath.FullName);
        }

        var openedFile = shouldCopyFile ? CopyTempPpt(filePath) : filePath;
        _tempPresentationFile = shouldCopyFile ? openedFile : null;

        var application = new Application();
        application.DisplayAlerts = PpAlertLevel.ppAlertsNone;
        //application.Visible = MsoTriState.msoFalse;

        try
        {
            Application = application;
            Presentation = application.Presentations.Open(openedFile.FullName + PasswordMark, MsoTriState.msoTrue,
                MsoTriState.msoFalse,
                MsoTriState.msoFalse);
        }
        catch
        {
            try
            {
                application.Quit();
                Marshal.ReleaseComObject(application);
            }
            catch (COMException ex)
            {
                Debug.WriteLine(ex);
            }

            Application = null;
            TryKillPowerPointProcess();
            throw;
        }
    }

    public IReadOnlyList<string> ExportSlideImages(string outputFolder, IProgress<string>? statusReporter, CancellationToken cancellationToken)
    {
        var presentation = Presentation ?? throw new InvalidOperationException("PowerPoint 文件尚未打开。");
        if (string.IsNullOrWhiteSpace(outputFolder))
        {
            throw new InvalidOperationException("输出目录不能为空。");
        }

        Directory.CreateDirectory(outputFolder);

        Slides? slides = null;
        try
        {
            slides = presentation.Slides;
            var slideCount = slides.Count;
            var slideWidth = Math.Max(1, (int) Math.Ceiling(presentation.PageSetup.SlideWidth * ExportDpi / 72d));
            var slideHeight = Math.Max(1, (int) Math.Ceiling(presentation.PageSetup.SlideHeight * ExportDpi / 72d));
            var slideImagePaths = new List<string>(slideCount);

            for (var slideIndex = 1; slideIndex <= slideCount; slideIndex++)
            {
                cancellationToken.ThrowIfCancellationRequested();
                statusReporter?.Report($"正在导出原始页面截图 ({slideIndex}/{slideCount})...");

                Slide? slide = null;
                try
                {
                    slide = slides[slideIndex];
                    var slideImagePath = Path.Join(outputFolder, $"{slideIndex:D3}.png");

                    // 换一个路径，防止写入失败
                    var tempImagePath = Path.Join(Path.GetTempPath(), $"{Path.GetRandomFileName()}.png");

                    slide.Export(tempImagePath, "PNG", slideWidth, slideHeight);
                    File.Copy(tempImagePath, slideImagePath, true);
                    File.Delete(tempImagePath);

                    slideImagePaths.Add(slideImagePath);
                }
                finally
                {
                    if (slide is not null)
                    {
                        Marshal.ReleaseComObject(slide);
                    }
                }
            }

            return slideImagePaths;
        }
        finally
        {
            if (slides is not null)
            {
                Marshal.ReleaseComObject(slides);
            }
        }
    }

    public static Task<IReadOnlyList<string>> ExportSlideImagesAsync(
        FileInfo filePath,
        string outputFolder,
        IProgress<string>? statusReporter,
        CancellationToken cancellationToken)
    {
        ArgumentNullException.ThrowIfNull(filePath);

        return RunOnStaThreadAsync(() =>
        {
            using var provider = new PowerPresentationProvider();
            provider.OpenPowerPresentation(filePath);
            return provider.ExportSlideImages(outputFolder, statusReporter, cancellationToken);
        }, cancellationToken);
    }

    private static Task<T> RunOnStaThreadAsync<T>(Func<T> action, CancellationToken cancellationToken)
    {
        ArgumentNullException.ThrowIfNull(action);

        if (cancellationToken.IsCancellationRequested)
        {
            return Task.FromCanceled<T>(cancellationToken);
        }

        var taskCompletionSource = new TaskCompletionSource<T>(TaskCreationOptions.RunContinuationsAsynchronously);
        var thread = new Thread(() =>
        {
            try
            {
                cancellationToken.ThrowIfCancellationRequested();

                var result = action();
                taskCompletionSource.TrySetResult(result);
            }
            catch (OperationCanceledException ex)
            {
                taskCompletionSource.TrySetCanceled(ex.CancellationToken.CanBeCanceled ? ex.CancellationToken : cancellationToken);
            }
            catch (Exception ex)
            {
                TryKillPowerPointProcess();
                taskCompletionSource.TrySetException(ex);
            }
        });

        thread.IsBackground = true;
        thread.SetApartmentState(ApartmentState.STA);
        thread.Start();
        return taskCompletionSource.Task;
    }

    private static void TryKillPowerPointProcess()
    {
        try
        {
            foreach (var process in Process.GetProcessesByName("powerpnt"))
            {
                try
                {
                    process.Kill();
                }
                catch (InvalidOperationException ex)
                {
                    Debug.WriteLine(ex);
                }
                catch (System.ComponentModel.Win32Exception ex)
                {
                    Debug.WriteLine(ex);
                }
            }
        }
        catch (InvalidOperationException ex)
        {
            Debug.WriteLine(ex);
        }
        catch (System.ComponentModel.Win32Exception ex)
        {
            Debug.WriteLine(ex);
        }
    }

    /// <summary>
    /// 拷贝一份临时Ppt 为什么需要复制，防止用户打开的时候也使用 WPS 打开
    /// </summary>
    /// <param name="pptFilePath"></param>
    /// <returns></returns>
    private static FileInfo CopyTempPpt(FileInfo pptFilePath)
    {
        var tempPptPath = new FileInfo(Path.Combine(Path.GetTempPath(), Path.GetRandomFileName() + pptFilePath.Extension));
        pptFilePath.CopyTo(tempPptPath.FullName);
        return tempPptPath;
    }

    #region IDisposable Support

    private void Dispose(bool disposing)
    {
        if (_disposedValue)
        {
            return;
        }

        var presentation = Presentation;
        var application = Application;
        Presentation = null;
        Application = null;

        try
        {
            if (presentation is not null)
            {
                presentation.Close();
                Marshal.ReleaseComObject(presentation);
            }

            if (application is not null)
            {
                application.Quit();
                Marshal.ReleaseComObject(application);
            }
        }
        catch (COMException ex)
        {
            Debug.WriteLine(ex);
        }
        catch (InvalidComObjectException ex)
        {
            Debug.WriteLine(ex);
        }

        if (_tempPresentationFile is not null)
        {
            try
            {
                if (_tempPresentationFile.Exists)
                {
                    _tempPresentationFile.Delete();
                }
            }
            catch (IOException ex)
            {
                Debug.WriteLine(ex);
            }
            catch (UnauthorizedAccessException ex)
            {
                Debug.WriteLine(ex);
            }

            _tempPresentationFile = null;
        }

        _disposedValue = true;
    }

    ~PowerPresentationProvider()
    {
        // 请勿更改此代码。将清理代码放入以上 Dispose(bool disposing) 中
        Dispose(false);
    }

    // 添加此代码以正确实现可处置模式。
    public void Dispose()
    {
        // 请勿更改此代码。将清理代码放入以上 Dispose(bool disposing) 中
        Dispose(true);
        // 使用了终结器，需要取消注释以下行
        GC.SuppressFinalize(this);
    }

    #endregion
}
```

这里需要注意 Interop 调用 PowerPoint 必须在 STA 线程运行，我封装了静态的 `ExportSlideImagesAsync` 方法自动处理线程切换，不需要手动管理线程。

第二个是 `PowerPointReader` 类，用 OpenXML 读取每一页的文本内容，包括标题、文本框、表格、备注，同时调用上面的截图导出方法，组装成结构化的幻灯片信息，代码如下：

```csharp
class PowerPointReader
{
    public async Task<IReadOnlyList<PowerPointSlideInfo>> ReadSlidesAsync(FileInfo pptxFile)
    {
        ArgumentNullException.ThrowIfNull(pptxFile);
        if (!pptxFile.Exists)
            throw new FileNotFoundException("PPTX 文件不存在", pptxFile.FullName);

        // 1. 读取所有幻灯片文本和结构
        var slideInfos = new List<(int Index, string Text)>();
        using (var presentation = PresentationDocument.Open(pptxFile.FullName, false))
        {
            var presentationPart = presentation.PresentationPart;
            if (presentationPart == null || presentationPart.Presentation == null)
                throw new InvalidOperationException("PPTX 文件结构无效");

            var slideIdList = presentationPart.Presentation.SlideIdList;
            if (slideIdList == null)
                throw new InvalidOperationException("未找到幻灯片列表");

            int slideIndex = 1;
            foreach (var slideId in slideIdList.Elements<SlideId>())
            {
                var relId = slideId.RelationshipId;
                var slidePart = (SlidePart)presentationPart.GetPartById(relId!);
                var slide = slidePart.Slide;
                var sb = new StringBuilder();

                // 标题
                var titleShape = slide.Descendants<Shape>().FirstOrDefault(s =>
                    s.ShapeProperties == null && s.TextBody != null &&
                    s.NonVisualShapeProperties?.NonVisualDrawingProperties?.Name?.Value?.Contains("标题") == true);
                if (titleShape != null)
                {
                    var titleText = GetShapeText(titleShape);
                    if (!string.IsNullOrWhiteSpace(titleText))
                        sb.AppendLine($"[标题] {titleText}");
                }

                // 所有文本框
                foreach (var shape in slide.Descendants<Shape>())
                {
                    var text = GetShapeText(shape);
                    if (string.IsNullOrWhiteSpace(text))
                        continue;

                    var isTitle = shape.NonVisualShapeProperties?.NonVisualDrawingProperties?.Name?.Value?.Contains("标题") == true;
                    if (!isTitle)
                        sb.AppendLine($"[文本框] {text}");
                }

                // 表格
                foreach (var table in slide.Descendants<DocumentFormat.OpenXml.Drawing.Table>())
                {
                    sb.AppendLine("[表格]");
                    foreach (var row in table.Descendants<DocumentFormat.OpenXml.Drawing.TableRow>())
                    {
                        var rowText = string.Join(" | ", row.Descendants<DocumentFormat.OpenXml.Drawing.TableCell>().Select(cell => GetParagraphsText(cell.TextBody)));
                        sb.AppendLine(rowText);
                    }
                }

                // 备注
                if (slidePart.NotesSlidePart != null)
                {
                    var notesSlide = slidePart.NotesSlidePart.NotesSlide;
                    // NotesSlide 的所有 Paragraph
                    var notesText = GetNotesSlideText(notesSlide);
                    if (!string.IsNullOrWhiteSpace(notesText))
                        sb.AppendLine($"[备注] {notesText}");
                }

                slideInfos.Add((slideIndex, sb.ToString().Trim()));
                slideIndex++;
            }
        }

        // 2. 导出每页截图
        var outputDir = Path.Combine(Path.GetTempPath(), $"pptx_{Guid.NewGuid():N}");
        var imagePaths = await PowerPresentationProvider.ExportSlideImagesAsync(pptxFile, outputDir, null, CancellationToken.None);

        // 3. 组装 PowerPointSlideInfo
        var result = new List<PowerPointSlideInfo>();
        for (int i = 0; i < slideInfos.Count; i++)
        {
            var slideIndex = slideInfos[i].Index;
            var slideText = slideInfos[i].Text;
            var imageFile = new FileInfo(imagePaths.Count > i ? imagePaths[i] : string.Empty);
            result.Add(new PowerPointSlideInfo(slideIndex, slideText, imageFile));
        }
        return result;
    }


    private static string GetShapeText(Shape shape)
    {
        // shape.TextBody 可能是 Presentation.TextBody，需遍历其所有 Drawing.Paragraph
        if (shape.TextBody == null) return string.Empty;
        var sb = new StringBuilder();
        foreach (var para in shape.TextBody.Descendants<DocumentFormat.OpenXml.Drawing.Paragraph>())
        {
            var text = string.Concat(para.Descendants<DocumentFormat.OpenXml.Drawing.Text>().Select(t => t.Text));
            if (!string.IsNullOrWhiteSpace(text))
            {
                sb.AppendLine(text);
            }
        }
        return sb.ToString().Trim();
    }

    private static string GetNotesSlideText(NotesSlide notesSlide)
    {
        if (notesSlide == null) return string.Empty;
        var sb = new StringBuilder();
        foreach (var para in notesSlide.Descendants<DocumentFormat.OpenXml.Drawing.Paragraph>())
        {
            var text = string.Concat(para.Descendants<DocumentFormat.OpenXml.Drawing.Text>().Select(t => t.Text));
            if (!string.IsNullOrWhiteSpace(text))
            {
                sb.AppendLine(text);
            }
        }
        return sb.ToString().Trim();
    }

    private static string GetParagraphsText(DocumentFormat.OpenXml.Drawing.TextBody? textBody)
    {
        if (textBody == null) return string.Empty;
        var sb = new StringBuilder();
        foreach (var para in textBody.Descendants<DocumentFormat.OpenXml.Drawing.Paragraph>())
        {
            var text = string.Concat(para.Descendants<DocumentFormat.OpenXml.Drawing.Text>().Select(t => t.Text));
            if (!string.IsNullOrWhiteSpace(text))
            {
                sb.AppendLine(text);
            }
        }
        return sb.ToString().Trim();
    }
}
```

这里仅支持 PPTX 格式的文档，如果需要兼容旧版 PPT 格式，可以先将文档转换成 PPTX 之后再处理。

## 核心多代理协作逻辑

整个流程的执行逻辑如下：

1. 初始化 MainAgent，让其根据要求生成初始的 SubAgent 提示词
2. 读取目标 PPT 的所有文本和每页截图
3. SubAgent 逐页分析 PPT，每一页的分析结果会作为上下文传给下一页的分析
4. 所有页面分析完成后，中立 Agent 对所有分析结果做整体评价
5. 结合中立 Agent 的评价和人类的输入评价，MainAgent 

核心实现全部在 Program.cs 中，以下分模块讲解：

### 1. 初始化大模型客户端和 MainAgent

```csharp
var keyFile = @"C:\lindexi\Work\Doubao.txt";
var key = File.ReadAllText(keyFile);

var openAiClient = new OpenAIClient(new ApiKeyCredential(key), new OpenAIClientOptions()
{
    Endpoint = new Uri("https://ark.cn-beijing.volces.com/api/v3"),
    NetworkTimeout = TimeSpan.FromHours(1), // 批量分析耗时久，设置长超时
});

var chatClient = openAiClient.GetChatClient("ep-20260306101224-c8mtg");
var pptPagePrompt = string.Empty;
var pageAnalysisResults = new List<(int PageNumber, string PageContains, string ContextRole)>();

ChatClientAgent mainAgent = chatClient.AsIChatClient()
    .AsBuilder()
    .BuildAIAgent(new ChatClientAgentOptions()
    {
        ChatOptions = new ChatOptions()
        {
            Tools =
            [
                AIFunctionFactory.Create(SavePptPageAnalystPrompt, "保存PPT页面分析子代理提示词", description: "保存可直接作为 System Prompt 使用的 PPT 页面分析子代理提示词"),
            ]
        }
    });
```

这里对接的是豆包的 Ark API 端点，你可以换成自己的大模型地址。MainAgent 绑定了保存提示词的工具，生成的提示词会直接存入变量供后续使用，避免自然语言输出的不稳定性。

### 2. 读取 PPT 全量内容

```csharp
var pptFilePath = @"C:\lindexi\Work\示例文档.pptx";

var powerPointReader = new PowerPointReader();
var powerPointSlideInfoList = await powerPointReader.ReadSlidesAsync(new FileInfo(pptFilePath));
var fullPptTextBuilder = new StringBuilder();
foreach (var powerPointSlideInfo in powerPointSlideInfoList)
{
    fullPptTextBuilder.AppendLine($"---第 {powerPointSlideInfo.SlideIndex} 页---");
    fullPptTextBuilder.AppendLine(powerPointSlideInfo.SlideText);
}
var fullPptText = fullPptTextBuilder.ToString();
```

我自己封装的 `PowerPointReader` 基于 OpenXML 实现，读取所有页面的文本内容并拼接成全量 PPT 文本，后续作为参数传给 SubAgent 做上下文判断。PPT 截图提取逻辑封装在 `PowerPresentationProvider` 中

### 3. 初始化 MainAgent 任务生成第一版提示词

```csharp
var mainSession = await mainAgent.CreateSessionAsync();

ChatMessage initializePromptEngineerMessage = new(ChatRole.System, $"""
你是一个提示词生成工程师。你的任务是编写并持续优化一个“PPT 页面分析子代理”的提示词。

子代理的固定输入：
1. 一整份 PPT 的全部文本，文本内容会明确标注页码。
2. 当前页面的文本。
3. 当前页面的截图。

子代理将被以下 C# 代码进行替换提示词内容，请确保你编写的提示词中包含正确的占位符，以便代码正确替换并传入相应内容：

var prompt = subAgentPrompt.Replace("$(AllPptText)", allPptText)
    .Replace("$(SlideIndex)", slideIndex.ToString())
    .Replace("$(CurrentPageText)", currentPageText)
    .Replace("$(PreviousResults)", previousResults);

// 省略剩余初始化提示词内容
""");

await ExecuteMainAgentAsync(initializePromptEngineerMessage, mainSession);

if (string.IsNullOrWhiteSpace(pptPagePrompt))
{
    throw new InvalidOperationException("主代理未生成 PPT 页面分析子代理提示词。");
}
```

这里给 MainAgent 明确了提示词的要求、占位符规则、输入输出规范，MainAgent 会直接生成第一版可直接使用的 SubAgent 提示词。

### 4. 核心迭代优化循环

```csharp
while (true)
{
    var previousResults = new StringBuilder();
    var subAgentResults = new List<(int SlideIndex, string PageContains, string ContextRole, string RawResponse)>();
    pageAnalysisResults.Clear();

    // 遍历所有页面调用 SubAgent 分析
    for (int i = 0; i < powerPointSlideInfoList.Count; i++)
    {
        var slideInfo = powerPointSlideInfoList[i];
        var screenshotPath = slideInfo.SlideImageFile.FullName;
        var prevResultsText = previousResults.ToString();

        var analysisResult = await AnalyzeCurrentPageAsync(pptPagePrompt, fullPptText, slideInfo.SlideIndex, slideInfo.SlideText, prevResultsText, screenshotPath);
        subAgentResults.Add((slideInfo.SlideIndex, analysisResult.PageContains, analysisResult.ContextRole, analysisResult.RawResponse));
        pageAnalysisResults.Add((slideInfo.SlideIndex, analysisResult.PageContains, analysisResult.ContextRole));

        // 累加之前页面的分析结果，供下一页做上下文判断
        previousResults.AppendLine($"---第 {slideInfo.SlideIndex} 页---");
        previousResults.AppendLine($"这页面包含了啥：{analysisResult.PageContains}");
        previousResults.AppendLine($"在页面上下文的作用：{analysisResult.ContextRole}");
    }

    // 中立 Agent 评价本轮 SubAgent 输出
    var neutralEvalPrompt =
        "你是一个中立的AI评估者。请根据以下所有页面的分析结果，评价SubAgent的整体表现，包括但不限于：事实描述的准确性、上下文作用分析的合理性、表达的清晰性、幻觉/编造内容等。请输出结构化评价意见。\n\n" +
        "所有页面分析结果：\n" + previousResults.ToString();
    var neutralAgent = chatClient.AsIChatClient().AsBuilder().BuildAIAgent();
    var neutralEvalSession = await neutralAgent.CreateSessionAsync();
    var neutralEvalUpdates = neutralAgent.RunStreamingAsync(new ChatMessage(ChatRole.User, neutralEvalPrompt), neutralEvalSession);
    var (_, neutralEvalResult) = await RunStreamingAsync(neutralEvalUpdates);

    // 支持人类补充评价
    Console.WriteLine();
    Console.WriteLine("请对本轮SubAgent表现进行评价（可空，直接回车跳过）：");
    var humanEval = Console.ReadLine() ?? string.Empty;

    // MainAgent 优化提示词
    var optimizePrompt =
        $@"请根据本次所有页面的执行表现、中立Agent的评价和人类评价继续优化子代理提示词，并通过工具输出新的完整系统提示词。

固定任务不要变化：
- 输入仍然是整份 PPT 文本、当前页文本、当前页截图。
- 输出仍然是“这页面包含了啥”和“在页面上下文的作用”两个维度。

本次所有页面分析结果：
{previousResults}
中立Agent评价：
{neutralEvalResult}
人类评价：
{humanEval}
优化重点：
- 继续提升“页面事实描述”和“上下文作用判断”的区分度。
- 继续强调忠实描述截图与文本，不要幻觉。
- 继续强调基于整份 PPT 文本理解当前页作用。";

    mainSession = await mainAgent.CreateSessionAsync();
    mainSession.SetInMemoryChatHistory([initializePromptEngineerMessage]);
    await ExecuteMainAgentAsync(new ChatMessage(ChatRole.User, optimizePrompt), mainSession);

    // 输出本轮结果
    Console.WriteLine();
    Console.WriteLine("---------");
    Console.WriteLine("本次优化生成的子代理提示词：");
    Console.WriteLine(pptPagePrompt);

    Console.WriteLine();
    Console.WriteLine("是否继续优化？输入 y 继续，其他任意键退出：");
    var input = Console.ReadLine();
    if (!string.Equals(input, "y", StringComparison.OrdinalIgnoreCase))
    {
        break;
    }
}
```

这里实现了完整的迭代闭环，每一轮都会跑完整份PPT的分析，然后中立Agent给出客观评价，同时支持人类补充评价，MainAgent会根据所有反馈优化提示词，你可以多轮迭代直到效果符合预期。

### 5. 单页 SubAgent 分析逻辑

```csharp
async Task<(string PageContains, string ContextRole, string RawResponse)> AnalyzeCurrentPageAsync(
    string subAgentPrompt,
    string allPptText,
    int slideIndex,
    string currentPageText,
    string previousResults,
    string screenshotPath)
{
    var pageContains = string.Empty;
    var contextRole = string.Empty;

    var tool = AIFunctionFactory.Create(SubmitPageAnalysis, "提交页面分析结果", description: "提交当前 PPT 页面分析结果，包含页面事实描述和页面上下文作用");

    // 可切换成本地 Ollama 部署的多模态模型
    var ollamaEndpoint = new Uri("http://172.20.113.28:11434");
    const string modelId = "qwen3-vl:8b";
    var ollamaApiClient = new OllamaApiClient(ollamaEndpoint, modelId);
    var pageAnalystAgent = ollamaApiClient.AsAIAgent(tools: [tool]);

    // 替换提示词占位符
    var prompt = subAgentPrompt.Replace("$(AllPptText)", allPptText)
        .Replace("$(SlideIndex)", slideIndex.ToString())
        .Replace("$(CurrentPageText)", currentPageText)
        .Replace("$(PreviousResults)", previousResults);

    var pageAnalystSession = await pageAnalystAgent.CreateSessionAsync();
    ChatMessage userMessage = new(ChatRole.User,
    [
        new TextContent(prompt),
        CreateScreenshotContent(screenshotPath)
    ]);

    var runResult = await ExecuteSubAgentAsync(pageAnalystAgent, pageAnalystSession, userMessage);

    if (string.IsNullOrWhiteSpace(pageContains))
    {
        pageContains = runResult.ContentText;
    }

    if (string.IsNullOrWhiteSpace(contextRole))
    {
        contextRole = "子代理未通过工具明确输出页面上下文作用，可根据直接回复继续优化提示词。";
    }

    return (pageContains, contextRole, runResult.ContentText);

    [Description("提交当前页面分析结果")]
    void SubmitPageAnalysis(
        [Description("维度1：这页面包含了啥，需要忠实描述页面中实际存在的元素")] string currentPageContains,
        [Description("维度2：这页面在整份 PPT 上下文中的作用")] string currentContextRole)
    {
        pageContains = currentPageContains;
        contextRole = currentContextRole;
    }
}
```

这里的SubAgent支持对接本地Ollama部署的多模态大模型，同时绑定了提交分析结果的工具，保证输出的结构化，避免自然语言回复的不稳定性。

剩余的辅助函数主要处理网络异常重试、图片内容封装、流式输出解析，逻辑比较简单就不展开讲解，完整代码可以在本文末尾找到下载方法。

## 注意事项

1. 提示词里的占位符`$(AllPptText)`、`$(SlideIndex)`等不能修改，否则代码里的替换逻辑会失效
2. 本地部署多模态模型的时候，建议选择支持图片输入的大模型，比如通义千问3-VL、Llava等，保证截图分析的准确性
3. 迭代的时候如果遇到幻觉问题，可以在人类评价里明确指出问题点，MainAgent会针对性优化提示词
4. 我自己迭代了3轮之后，SubAgent的分析准确率已经达到了90%以上，完全可以满足批量PPT分析的需求
5. 引入本地 ollama 对接千问 qwen3-vl:8b 只是为了降低成本，全走豆包的话，平均一页 PPT 消耗 Token 在一万左右，成本不低

本文代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/2638d2f3aa3c9aa8191c4d84501d6b9feddde82d/SemanticKernelSamples/JiyinunalcheWaqerehoqarlijear) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/blob/2638d2f3aa3c9aa8191c4d84501d6b9feddde82d/SemanticKernelSamples/JiyinunalcheWaqerehoqarlijear) 上，可以使用如下命令行拉取代码。我整个代码仓库比较庞大，使用以下命令行可以进行部分拉取，拉取速度比较快

先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin 2638d2f3aa3c9aa8191c4d84501d6b9feddde82d
```

以上使用的是国内的 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码，将 gitee 源换成 github 源进行拉取代码。如果依然拉取不到代码，可以发邮件向我要代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin 2638d2f3aa3c9aa8191c4d84501d6b9feddde82d
```

获取代码之后，进入 SemanticKernelSamples/JiyinunalcheWaqerehoqarlijear 文件夹，即可获取到源代码

更多技术博客，请参阅 [博客导航](https://blog.lindexi.com/post/%E5%8D%9A%E5%AE%A2%E5%AF%BC%E8%88%AA.html )

