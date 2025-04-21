
# GPT 炼丹记录

本文记录我使用 GPT 测试的内容

<!--more-->


<!-- CreateTime:2023/10/10 10:37:09 -->

<!-- 不发布 -->

## 润色

```csharp
我正在制作一个软件的提示文案内容，我需要你帮忙润色我给出的文案，要求润色之后的文案内容意思不变同时通俗易懂，且去掉口语化的表述。
不好意思，你输入的内容触发了内容风险监控，请修改你的输入内容之后重试
```

```
我正在编写一篇文章，我只有构成这篇文章的一些小点。我需要你帮忙将我给出的小点串联起来形成一篇文章，要求形成的文章通俗易懂且去掉口语化的表述，你可以打乱我给出的小点的顺序以获取更加通顺的文章描述

以下是我给出的小点：

- 现在教师疲惫于备课
- 使用 AIGC 帮助教师备课是趋势
- 教师备课的课件在低年级需要丰富的图片和音视频等多媒体
- 教师备课的课件在高年级需要用到公式，比如数学和物理化学等
- 通过 AIGC 多模态生成的资源不满足教学需求。原因是不贴合教材
- 相同的资源可能被用在不同的课程里面，而 AIGC 无法识别。比如一张酒杯的图片，可能在语文里是用来上 “举杯邀明月” ，而在生物课程里是用来上微生物发酵，难以让 AIGC 识别或生成满足需求的图片
- 现在的 AIGC 对视频的生成能力较弱且成本较高，在教育领域对视频的要求较高，而 AIGC 生成的视频可能无法满足管控
- 现在的课堂上有些老师喜欢使用互动课堂授课模式，互动课堂课件的元素是自定义的，难以被 AIGC 理解和生成

编写的文章：
```

```
你是一个审稿人，现在我有一段文本给到你，你需要帮我分析这段文本里面是否有错别字内容

---

本文先从项目搭建开始告诉大家如何创建一个源代码生成器项目。本文后续的内容将会在这个项目中进行演示。本文的编写顺序是先搭建项目，然后再讲解一些基础的概念和用法，再到如何进行调试，最后提供一些实际的演练给到大家。基础知识部分也放在演练里面，先做演练再讲基础知识，防止一口气拍出大量基础知识劝退大家
```


## 翻译

```
请帮我将以下内容进行总结然后翻译成英文：

我统计了我的应用，根据我从 3016336 位用户过去 7 天的数据里面，可以看到平均的启动耗时是 2.7 秒。这里需要进一步说明的是我的应用是一个复杂的应用，启动过程是相当复杂的
```

```
请将我的博客翻译为英文，你可以修改写作的风格，以及调整语句让文章总体更具备可读性
```

```
我正在 github 上报告一个问题，请根据以下内容帮我拟定一个英文标题和内容，要求标题简略：

[Content Start]

在 WPF 里面，如果使用了 ManagementEventWatcher 监听 WMI 变更，则会导致触摸失效

复现步骤如下

1. 安装 System.Management 库用于使用 WqlEventQuery 监听 WMI 变更
2. 监听 TouchDown 事件输出断点信息

整个代码如下

[Code1]

当我注释掉 `insertWatcher.Start();` 这行代码的时候，我触摸我的窗口，我可以看到 `MainWindow_TouchDown` 方法的断点被命中。当我执行过 `insertWatcher.Start();` 这行代码的时候，无论我如何触摸窗口，都不会进入 `MainWindow_TouchDown` 方法

且此时可以看到 FirstChanceException 事件被触发，打印的异常信息如下

[Code2]

我所使用的代码的 MainWindow 文件内容如下

[Code3]

我将我的最小复现项目上传到 github 上，你可以拉取我的项目复现问题

[Content End]

```

```
我正在 github 上分享我的成果，请根据以下内容帮我拟定英文标题和内容，要求标题简略，内容符合程序员风格，语气尽量激动，你可以修改内容顺序使逻辑更清晰：

[Content Start]

我制作了一个名为 UnoSpySnoop 的工具。这个 UnoSpySnoop 是一款用来辅助调试 Skia 平台下的 UNO 应用 UI 界面的工具。

为什么需要此工具？

因为在 Skia 平台下，无论是 WPF 还是 GTK 都采用一个 Surface 来渲染界面。这就导致了原本的 WPF 的 UI 调试工具，如 SnoopWpf 或 LiveVisualTree 等工具，将只能看到一张图片而不能获取正确的界面结构。通过 UnoSpySnoop 可以很好的在基于 Skia 的桌面平台，如 Skia.Wpf 和 Skia.Gtk 上，进行辅助界面开发调试，提高开发者的界面开发效率，特别是调试在 Linux 桌面上的 Skia.Gtk 应用的时候

此工具的实现了类似于 《Inspecting the runtime visual tree of an Uno app》 文档介绍里面的 UWP 应用的内置在 VisualStudio 的 LiveVisualTree 工具的功能，此工具可以协助开发者了解 Skia 平台的应用的界面结构，可以获取某个 UIEelement 的各个属性

此工具在 GitHub 上使用最友好的 MIT 协议进行开源，请看：[Link1]

我希望 UnoSpySnoop 工具能够帮助开发者真正提高 Skia 桌面平台应用的开发调试效率。如果你有任何的建议和问题我都十分欢迎收到反馈

再次感谢 UNO 平台，感谢 UNO 的贡献者们

[Content End]
```

```
我正在 github 上编写文档，请根据以下内容帮我编写符合程序员风格的英文内容：

[Content Start]

根据我之前的调查，这是一个设备丢失问题。开启缓存之后，当设备丢失之后，重新创建新设备时，没有让旧设备关联的缓存失效，从而导致缓存跨设备使用，进而导致了渲染问题。这个问题是在比较底层的渲染模块发生的，我没有调查到具体的代码

[Content End]

英文内容：
```

```
我正在 github 上新建一个 PR 请根据以下内容帮我编写符合程序员风格的英文标题和内容，要求标题简略：

[Content Start]

优化了 SKRoundRectCache 的 Clear 方法，在 dotnet 6 以及更高版本上，使用更加快速的清理方法

[Content End]

英文标题：
```

```
请帮我将以下内容转述为地道的英文：事实上，对于 GFX 层上的代码更改是超过我的能力的，甚至我担心更改会导致更多更加严重的问题。所以我只能提供一些妥协的方案
```

```
请帮我将以下内容转述为地道的英文：由于 `Dictionary` 的容量是成倍增加的，因此不是 2 的指数倍数的 300 数值，我认为 300 这个数值是不合理的，存在一些空间上的浪费
```

```
请帮我将以下内容转述为地道的计算机英文：我认为如果能够将 C# 代码和 XAML 进行混合，那将会是多么有趣的事情，就像 Razor 一样。可以允许在 XAML 里面内插 C# 代码。内插的 C# 代码可以用来做很多繁琐的转换工作。回忆一下，如果咱需要在 XAML 里面写一个数据转换器，咱需要几个步骤？第一步，创建一个 ValueConverter 在 C# 代码里面。第二步，在资源里面定义转换器。第三步，在需要使用转换器的地方引用资源。但很多情况下，按照我的经验，我都是仅有单次的转换数据的需求，这就意味着刚才创建的 ValueConverter 转换器只为这一处 XAML 代码服务，但我不得不引入大量的 XAML 和 C# 代码
```

```
请帮我将以下内容转述为地道的计算机英文：这是一个令人期待的功能。随着 NuGet 包的数量越来越多，现在开发者可能需要花费很多的精力和时间投入到寻找一个合适自己的 NuGet 包。如果能够在 AI 的帮助下，简化这一个过程，那无疑能够促进 dotnet 的生态发展
```

```
我想要写信报告一个问题，请你帮我拟一个报告问题的英文标题和一个报告问题的英文内容，要求标题简略，你可以重新组织内容以让内容更加通顺。

以下是我想要报告的问题：

在 OpenXML 里面，设置 PresentationDocument 的 PackageProperties 的 Title 属性为包含 `"\u0001"` 的字符串时，将会提示 ArgumentException 错误

最简的复现代码如下

[Code1]

运行以上代码，将会出现如下错误

[Code2]
```


```
你是一个专业英文文档撰写员，请根据以下中文文档，撰写计算机专业的英文文档

[Content Start]

[Content End]
```

```
你是一个专业英文文档撰写员，请根据以下中文内容，撰写计算机专业的英文 Show and tell 文档。文档包含标题和内容。你可以修改和添加内容使文档更加流利，要求语气激动，内容富有激情

[Content Start]

我想要和你分享我最近的研究内容。我在去年的时候就反馈过 Avalonia 的渲染延迟问题，在今年开始，我尝试为一个 Avalonia 应用构建笔迹书写模块。然而坏消息是，无论我如何努力，我都无法让 Avalonia 的渲染延迟达到 WPF 的水平。经过和我的伙伴们讨论，我决定让 WPF 作为 Avalonia 的笔迹加速层。

核心思想就是在现有的 Avalonia 应用的上方放一个 WPF 的透明窗口，让 WPF 窗口绘制笔迹的内容。当用户抬手的时候，再执行 Wet->Dry 过程，将笔迹元素从 WPF 窗口移除然后在 Avalonia 窗口里面显示

实现以上思想的代码被我放在 GitHub 上：[link1]

我十分期望 Avalonia 能够优化渲染延迟。如果有任何我能帮忙的，我会十分乐意花费我的精力和时间在这里

[Content End]

Title: 
```

```
我正在给 GPT 编写提示词 Prompt 内容，请帮我将以下的提示词进行优化和翻译为英文：
你是一个意图识别机器人，可以识别出用户说话的意图是什么。请根据用户输入的内容，将用户输入的内容分类为以下类别中的一类。如果用户输入的内容无法进行归类，则归类为未知。请只对用户输入内容的意图进行归类，不要回答任何问题。只能使用以下意图类别，不能添加新的意图类别
```


## 写通知

```csharp
请帮我写一篇通知，主题是《强化对未成年人使用的智能设备信息内容管理，全面清理违法不良信息》
```

## 代码命名

```csharp
我正在编写一个功能，这个功能是 对当前正在排版的文本内容进行回溯，每一段每一行都需要进行回溯。我正在编写的是 C# 代码，请为这个功能起一个方法名，请给出多个候选命名给我选择
```

```csharp
我正在编写一个功能，这个功能是提供API约束，更具体的是约束类型里面的成员。我需要给这个功能编写一个类型，我正在编写的是 C# 代码，我需要你给出这个类型的命名，请给出多个候选命名给我选择
```

```csharp
我正在编写一个长时间运行的工作服务，我现在需要为这个服务进行命名。现在已知的是这个服务是使用 C# 代码编写的，需要你给出符合 C# 的项目的命名空间命名规范的服务名
```

```csharp
我正在编写一个功能，这个功能是将图片设为每页水印。我需要为此功能创建一个文件夹，我将在这个文件夹里面存放实现此功能的代码，我正在编写的是 C# 代码，我需要你给出这个文件夹的命名，请给出多个候选命名给我选择
```

```csharp
我正在编写一个功能，这个功能是一个表示“可被子类继承的内容”的属性。我正在编写的是 C# 代码，起一个符合 C# 命名规范的属性名，请给出多个候选命名给我选择
```

```csharp
我正在编写一个功能，这个功能是一个表示“坐标系”类型，要求这个类型名包含“Coord”词根。我正在编写的是 C# 代码，起一个符合 C# 命名规范的类型名，请给出多个候选命名给我选择
```

```csharp
我正在编写一个功能，这个功能是一个生成页面配图效果。我正在编写的是 C# 代码，我需要编写一个类型，这个类型表示生成页面配图里面的配图这个功能，我需要你给出配图这个功能的类型名，请给出多个候选命名给我选择，以及这些候选命名的理由

如果单纯是“配图”，你会用哪些命名？请同时给出理由
```

## 文转图

```csharp
请根据以下主题扩写一段风景的描述内容：古风的小路
```

```
我正在使用根据文字生成图片的功能，我需要你帮我将以下文本内容转换为适合于 DALL·E 模型生成图片的提示词内容：举杯邀明月，对影成三人
一个唐朝的诗人拿着酒杯看着月亮
```

```
你是一个根据文本生成图片的提示词工程师。我正在准备为一份教学使用的课件生成 PPT 的背景图，现在我需要你根据以下的课件内容输出用于根据文本生成图片的提示词文本。
举杯邀明月，对影成三人
```

## 生成思维导图


```
请提炼文本的主要内容，分多个层次，制作成 Markdown 格式的思维导图，要求一级节点只能有一个。例子如下：

输入：
#####
自主阅读文言文，读准字音，读通句子。理解： 品读重点句，借助注释理解课文内容，张开想象，加深理解。归纳：结合人物对话和表现人物动作、神态的词句，想象情境，用自己的话讲述故事。
#####

输出：
- 学法导航
  - 自主阅读
    - 读准字音
    - 读通句子
  - 理解课文内容
    - 张开想象
    - 加深理解
  - 归纳故事
    - 结合内容
    - 想象情境

输入：
#####
1.试着用自己的话说一说故事的意思。
2.小组内练讲故事。
3.班内展示。
#####

输出：
```

```csharp
- 故事讲述
  - 自我表达
    - 用自己的话解释故事
  - 小组活动
    - 小组内练习讲故事
  - 班级活动
    - 在班级内展示讲故事
```

## 点名程序

```
我需要你帮忙将以下输入内容进行拆分，以下输入内容是一批名单，请将名单拆分为一行一个名字。如果输入的内容不是名单，请输出拆分失败

[输入开始]
明明，小红，大黄，灏杰,睿兮,少妍,文琪.婧怡。子意。馨予；洁吉,夏洛,文语
,亦骏,清颜,熙嫣,丽媛,靓宇,成正,果沐,子灏,绎嫣,
美玥,俊朝,蓓颖,浩然,元景,可芯,芊泽,

锦溪,诗雅,诗雅,思皓,昕雅,柯钦,杰浩,若琳,姝钰,东蕾,天青,沐澄,思淳,文杰,炎彬,鑫瑜,佳哲,依琦,晨皓,轶涵,楷欣,一彬,宏玮,欣俪,安颐,语妍,玥硕,伟诚,云杰,乐颜,政阳,毅博,廷佳,麟萱,允淑,弈霖,家妤,文卿,至善,致远,锦怡
[输入结束]

拆分的名单：
```

```
我需要你帮忙将以下输入内容进行拆分，以下输入内容是一批词语，请将名单拆分为一行一个词语。如果输入的内容不是一个个词语，请输出拆分失败
[输入开始]
{{$Input}}
[输入结束]

拆分的词语：
```

## 排版界面

```csharp
如果你想让GPT理解“对称原理、轴心式布局、左右分布布局、上下分布布局”等布局原理，可以使用以下提示词：

- 角色：设计师
- 上下文：你正在为一个PPT设计布局
- 指令：请根据以下布局原理设计PPT页面：对称原理、轴心式布局、左右分布布局、上下分布布局
- 风格：简洁明了
- 输出格式：PPT页面


源: 与必应的对话， 2023/11/9
(1) 爆肝一星期，整理了这份简单易用的gpt提示词指南 - 知乎. https://zhuanlan.zhihu.com/p/640598049.
(2) 爆火ChatGPT代码解释器食用指南，来了 - 知乎 - 知乎专栏. https://zhuanlan.zhihu.com/p/642420229.
(3) 提问的艺术：如何通过提示词让 ChatGPT 更准确地理解你的问题？-腾讯云开发者社区-腾讯云. https://cloud.tencent.com/developer/article/2277818.
(4) 用魔法打败魔法！教你让gpt自己产生提示词 - 掘金. https://juejin.cn/post/7287494827702337536.
(5) undefined. https://www.learnprompt.org/.
```

当你为PPT设计布局时，作为设计师，你可以根据以下布局原理设计PPT页面：对称原理、轴心式布局、左右分布布局、上下分布布局。请以简洁明了的风格输出PPT页面。希望这些提示词能够帮助你！

```
当你为PPT设计布局时，作为设计师，你可以根据以下布局原理设计PPT页面：对称原理、轴心式布局、左右分布布局、上下分布布局。请以简洁明了的风格输出PPT页面。

以下是原始的未排版之前的 PPT 文档内容

#####

{
  "SlideIndex": 1,
  "SlideId": "b1caa0d40c8a459dbd2fd7596f83f6d5",
  "SlideWidth": 1280.0,
  "SlideHeight": 720.0,
  "Elements": [{
    "ElementId": "1999daba5ec64e02b877a80cce4060fc",
    "ElementType": "Picture",
    "SlideX": 148.153,
    "SlideY": 59.419,
    "Width": 372.12199999999996,
    "Height": 512.0
  }, {
    "ElementId": "3cf72c5fe2854a63baa0b2bcb5331c59",
    "ElementType": "Picture",
    "SlideX": 669.313,
    "SlideY": 59.39,
    "Width": 409.27200000000005,
    "Height": 512.012
  }, {
    "ElementId": "7a43186986114dd0bb441741a059a1e9",
    "ElementType": "Text",
    "SlideX": 985.8995,
    "SlideY": 33.70633333333333,
    "Width": 80.0,
    "Height": 52.79333333333334,
    "ElementContent": "古诗",
    "RichTextInfo": {
      "ParagraphInfoList": [{
        "RunPropertyInfoList": [{
          "FontName": "微软雅黑",
          "FontSizePixel": 39.99999999,
          "StartOffset": 0,
          "EndOffset": 1,
          "Length": 2,
          "Text": "古诗"
        }]
      }]
    }
  }, {
    "ElementId": "08cc99014c2140d7abc3420e041db1ed",
    "ElementType": "Text",
    "SlideX": 473.171,
    "SlideY": 36.14383333333333,
    "Width": 160.00000000000006,
    "Height": 52.79333333333334,
    "ElementContent": "理想信念",
    "RichTextInfo": {
      "ParagraphInfoList": [{
        "RunPropertyInfoList": [{
          "FontName": "微软雅黑",
          "FontSizePixel": 39.99999999,
          "StartOffset": 0,
          "EndOffset": 3,
          "Length": 4,
          "Text": "理想信念"
        }]
      }]
    }
  }, {
    "ElementId": "f15c8060-939f-4b84-af7c-0e20d85215c2",
    "ElementType": "Text",
    "SlideX": 965.9295,
    "SlideY": 271.65133333333335,
    "Width": 120.0,
    "Height": 52.79333333333335,
    "ElementContent": "回忆录",
    "RichTextInfo": {
      "ParagraphInfoList": [{
        "RunPropertyInfoList": [{
          "FontName": "微软雅黑",
          "FontSizePixel": 39.99999999,
          "StartOffset": 0,
          "EndOffset": 2,
          "Length": 3,
          "Text": "回忆录"
        }]
      }]
    }
  }]
}

#####

请在内容不变的前提下，对以上的 PPT 文档内容进行排版，请输出排版之后的结果
```


```
我给你一个点集，请根据点集猜出这些点集构建成的文字

#####

#####
```

## 代码审查

```
你是一位代码审查者，以下是一些 WPF 的 XAML 的代码，你需要审查出代码里面的不匹配问题。不匹配问题指的是 SolidColorBrush 所使用的 Color 和 
```

```
你是一位代码审查者，以下是一些 C# 的代码，你需要审查出代码里面的有哪些地方可以为其编写单元测试。请列举出可以编写单元测试的点

#####
public partial class TextEditorCore
{
    /// <summary>
    /// 追加一段文本，追加的文本按照段末的样式
    /// </summary>
    /// 这是对外调用的，非框架内使用
    [TextEditorPublicAPI]
    public void AppendText(string text)
    {
        if (string.IsNullOrEmpty(text))
        {
            return;
        }

        DocumentManager.AppendText(new TextRun(text));
    }

    /// <summary>
    /// 追加一段文本
    /// </summary>
    /// <param name="run"></param>
    /// 这是对外调用的，非框架内使用
    [TextEditorPublicAPI]
    public void AppendRun(IImmutableRun run)
    {
        DocumentManager.AppendText(run);
    }

    /// <summary>
    /// 在当前的文本上编辑且替换。文本没有选择时，将在当前光标后面加入文本。文本有选择时，替换选择内容为输入内容
    /// </summary>
    /// <param name="text"></param>
    /// <param name="selection">传入空时，将采用 <see cref="CurrentSelection"/> 当前选择范围</param>
    /// 这是对外调用的，非框架内使用
    [TextEditorPublicAPI]
    public void EditAndReplace(string text, Selection? selection = null)
    {
        AddLayoutReason("TextEditorCore.EditAndReplace(string text)");

        TextEditorCore textEditor = this;
        DocumentManager documentManager = textEditor.DocumentManager;
        // 判断光标是否在文档末尾，且没有选择内容
        var currentSelection = selection ?? CaretManager.CurrentSelection;
        var caretOffset = currentSelection.FrontOffset;
        var isEmptyText = string.IsNullOrEmpty(text);
        if (currentSelection.IsEmpty && caretOffset.Offset == documentManager.CharCount)
        {
            if (!isEmptyText)
            {
                // 在末尾，调用追加，性能更好
                documentManager.AppendText(new TextRun(text));
            }
        }
        else
        {
            if (isEmptyText)
            {
                documentManager.EditAndReplaceRun(currentSelection, null);
            }
            else
            {
                var textRun = new TextRun(text);
                documentManager.EditAndReplaceRun(currentSelection, textRun);
            }
        }
    }
}
#####

可以编写单元测试的内容：
```

## 询问问题

```
请编写 deb 安装包脚本，要求实现在 deb 安装包安装完成之后，自动启动所安装的程序，避免启动的应用发生 XOpenDisplay 错误。postinst.sh 脚本代码：

包含设置正确的 DISPLAY 的 postinst.sh 脚本代码：

以上代码哪里设置正确的 DISPLAY ？

请给出正确设置的 DISPLAY 的脚本，只给出设置 DISPLAY 的脚本，不要回答其他内容！！！

#!/bin/sh
set -e

# 设置 DISPLAY 环境变量为本地主机上的第一个显示和屏幕
export DISPLAY=:0.0

上述代码的 set -e 是什么意思

上述代码的 export DISPLAY=:0.0 是什么意思

如果此时 X server 没有运行，可以如何进行检测。检测 X server 运行情况的脚本是：

#!/bin/bash
set -e

# 检查 X server 是否运行
if ! xset q &>/dev/null; then
    echo "No X server at \$DISPLAY [$DISPLAY]" >&2
    exit 1
fi

继续给出使用 xdpyinfo 和 xprop 检测 X server 运行情况的脚本

deb 安装完成之后，启动安装的应用。安装的应用放在 /opt/apps/foo 文件夹，应用可执行文件是 fx ，应用是基于 X11 的，需要显示窗口。要求在 deb 安装之后，能够自动启动 fx 进程，使用普通用户身份运行，且让 fx 进程显示出窗口。postinst.sh 脚本代码：
```

debian 的 buster 是什么意思？


```
SYSTEM
Use the following context as your learned knowledge, inside <context></context> XML tags.

<context>

Avalonia 11.1 已知问题 IterationCount 为 Infinite 的动画播放出现异常

</context>

When answer to user:
- If you don't know, just say that you don't know.
- If you don't know when you are not sure, ask for clarification.
Avoid mentioning that you obtained the information from the context.
And answer according to the language of the user's question.

USER
Avalonia 动画为何失败

ASSISTANT
```


### 故事生成
请继续以下故事：
“从前有个小猫，……”




<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。