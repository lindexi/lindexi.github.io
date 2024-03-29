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

我发现在 Skia.Gtk 平台下，在 OnLaunched 里立刻使用 MainWindow.SetBackground 方法设置背景色是无效的，但如果进行一些延迟之后再设置则是有效的。这个行为和在 Skia.WPF 里面是不相同的，你可以运行我的 Demo 代码，分别切换 Skia.Gtk 和 Skia.Wpf 平台，查看其差异

[Content End]

英文标题：
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

咱可以发现在 Microsoft.Maui.Graphics.Win2D 等实现上，在 PlatformDrawPath 和 FillPath 等方法上，从 PathF 获取到平台相关的 Path 或 Geometry 时，将会依赖 PathF.PlatformPath 属性作为缓存，如此可以提高重复绘制的性能，减少创建平台相关的对象的次数

但是在 Microsoft.Maui.Graphics.Skia 上，每次都是重新创建 SKPath 对象，这就意味着重复绘制一个复杂的 PathF 将不能在 Microsoft.Maui.Graphics.Skia 获得较好的性能

本次更改的作用就是让 Microsoft.Maui.Graphics.Skia 利用 PathF.PlatformPath 属性作为缓存，提高 Microsoft.Maui.Graphics.Skia 重复绘制复杂 PathF 的性能

[Content End]

英文内容：
```

```
我正在 github 上新建一个 PR 请根据以下内容帮我编写符合程序员风格的英文标题和内容，要求标题简略：

[Content Start]

修复在 Skia.Gtk 平台下，在 OnLaunched 里立刻使用 MainWindow.SetBackground 方法设置背景色是无效的问题

问题原因是在 OnLaunched 里立刻使用 MainWindow.SetBackground 方法时，进入的 UnoGtkWindowHost.UpdateRendererBackground 里面，判断 `_renderer` 字段还没初始化，于是什么都不做。等待 InitializeAsync 执行 GtkRendererProvider.CreateForHostAsync 方法之后，才能获取到 `_renderer` 字段的值，然而此时将错过背景赋值逻辑

修复方法是在 GtkRendererProvider.CreateForHostAsync 方法执行完成之后，补充对 `_renderer.BackgroundColor` 赋值

[Content End]

英文标题：
```

```
请帮我将以下内容转述为地道的英文：

好消息是现在已经有许多的中国的开发者已经知道了 UNO ，而且有许多开发者进行尝试 UNO 框架了。其中开发者们集中使用的是 GTK 版本，用于开发 Linux 系统的桌面客户端应用。附带开发者们也会使用对应的 WPF 框架，用于顺带构建出 Windows 系统的桌面客户端应用。为什么不使用 WinUI 呢？核心原因是当前在中国仍有极大量的系统是 Windows 7 系统，无法使用 WinUI 框架，其次的原因是 WinUI 3 的稳定性很差。我创建了两个本地社区群组，用来交流 UNO 的开发经验，我编写了超过 10 篇关于 UNO 的开发博客。我也会持续的向我的本地社区宣传 UNO 框架，以及帮助他们解决开发问题。我认为，只要 UNO 能做好自己，包括做好框架功能的稳定性、完整性，优化开发者的开发调试体验，适配好各个平台，完善开发者文档，那么 UNO 自然就能收获许多的中国的开发者
```

```
请帮我将以下内容转述为地道的英文：

感谢你的建议。但是我认为 API 有分层也是一个好的设计，对于一些底层的，可能造成危险的 API 可以适当将其制作的难用一些，或使用方法需要阅读文档。如此可以尽量让使用方去阅读文档或了解更多的知识。当然，对于一些常用的 API 或更接近业务层的 API 我是非常同意让其更便于使用的
```

```
请帮我将以下内容转述为地道的英文：

在阅读了 UNO 的源代码之后，我认为咱可以将原本的只从 Mouse 里面获取事件，更改为同时从 Stylus 和 Mouse 获取事件。且在 Mouse 里面判断当前的事件是来自于 Stylus 或 Touch 时，将自动忽略后续处理逻辑。根据 WPF 的触摸行为可以知道，触摸先触发 Touch 事件，随后提升为 Stylus 事件，最后提升为 Mouse 事件。于此同时 Pen 消息将会先触发 Stylus 事件，再提升为 Mouse 事件。这就意味着简单处理的话，直接使用 Stylus 事件即可同时处理 Touch 和 Pen 的情况。使用 Stylus 事件时，将可以自动支持多指触摸。且可以通过 GetPropertyValue 方法，传入 StylusPointProperties.Width 和 StylusPointProperties.Height 获取到可能存在的触摸的尺寸，用于填充实现 Microsoft.UI.Input.PointerPointProperties.ContactRect 属性。这将有利于触摸屏上面的 WPF 应用使用上 UNO 框架
```

```
请帮我将以下内容转述为地道的英文：

不断的计算当前鼠标落点和上个鼠标落点之间构建的几何形状，然后通过 Geometry.Combine 方式与当前 Clip 采用 GeometryCombineMode.Exclude 方式叠加，如果即可让裁剪部分源源不断减去当前鼠标的轨迹。鼠标轨迹如下图，就是两个圆形和矩形的拼接
```

```
我想要写信报告一个问题，请你帮我拟一个报告问题的英文标题和一个报告问题的英文内容，要求标题简略，你可以重新组织内容以让内容更加通顺。

以下是我想要报告的问题：

当前行为： 当我使用 SKXamlCanvas 时，如果我在 PaintSurface 事件里面抛出任何异常，且当前的 PaintSurface 事件是由后台线程触发的，那将导致我的进程崩溃

预期行为：即使在 PaintSurface 事件里面抛出任何异常，应用程序也可以正常工作且收集到异常，比如通过 TaskScheduler.UnobservedTaskException 事件收集到异常

复现步骤：

1. 添加 SKXamlCanvas 到 xaml 里
2. 订阅 SKXamlCanvas 的 PaintSurface 事件，且在事件实现方法抛出异常
3. 在后台线程调用 SKXamlCanvas 的 Invalidate 方法

https://github.com/lindexi/lindexi_gd/tree/dde76effc23ebb9ee974b6ec276b242c39a50bdf/JagobawearjiNeewhiqakerki
```

## 写通知

```csharp
请帮我写一篇通知，主题是《强化对未成年人使用的智能设备信息内容管理，全面清理违法不良信息》
```

## 代码命名

```csharp
我正在编写一个功能，这个功能是表示 导入的 PPTX 文件被加密的异常。我正在编写的是 C# 代码，请为这个功能起一个异常类型名，请给出多个候选命名给我选择
```

```csharp
我正在编写一个功能，这个功能是根据输入的文件的文件后缀名决定文件是否被采纳。我需要将这个功能封装到一个继承 IFileImportFilter 接口的类型里面，我正在编写的是 C# 代码，我需要你给出这个类型的命名
```

```csharp
我正在编写一个长时间运行的工作服务，我现在需要为这个服务进行命名。现在已知的是这个服务是使用 C# 代码编写的，需要你给出符合 C# 的项目的命名空间命名规范的服务名
```

```csharp
我正在编写一个功能，这个功能是一个带特效的笔迹效果。我需要为此功能创建一个文件夹，我将在这个文件夹里面存放实现此功能的代码，我正在编写的是 C# 代码，我需要你给出这个文件夹的命名，请给出多个候选命名给我选择
```

```csharp
我正在编写一个功能，这个功能是一个表示“点和线的关系”属性。我正在编写的是 C# 代码，属性起一个符合 C# 命名规范的属性名，请给出多个候选命名给我选择
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