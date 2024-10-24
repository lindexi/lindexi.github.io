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

使用 `_x11.Display` 而不是 `_x11.DeferredDisplay` 的原因是 XSHM 需要推送到能够接收 Event 的 Display 上，以获取渲染完成 XShmCompletionEvent 事件

[Content End]

英文内容：
```

```
我正在 github 上新建一个 PR 请根据以下内容帮我编写符合程序员风格的英文标题和内容，要求标题简略：

[Content Start]

使用MIT-SHM优化渲染推送性能

## What does the pull request do?

此 pull request 实现了使用 MIT-SHM 的方式推送渲染的界面画面图片，使用了XShmPutImage方法代替了XPutImage方法。可以有效减少 Bitmap 在 X11 层的拷贝，提升推送渲染性能

## What is the current behavior?

当前只使用了 XPutImage 进行软渲染时推送界面图片

## What is the updated/expected behavior with this PR?

此 pull request 提供了更多选项，允许开发者配置使用 MIT-SHM 共享内存方式推送渲染图片，可以有效在低性能设备上提升渲染性能，降低渲染延迟

## How was the solution implemented (if it's not obvious)?

实现方法是通过 MIT-SHM 进行渲染图片的推送，按照 @kekekeks 在 Link1 提及的方法实现

[Content End]

英文标题：
```

```
请帮我将以下内容转述为地道的英文：

特别地，允许比最大渲染帧多一帧。这是因为可能可能现在正在有一帧正在返回中，一帧正在渲染中，所以再添加一帧进行准备渲染是能够最大化渲染效率的
```

```
请帮我将以下内容转述为地道的英文：

你所见的 WPF 部分占用了大量的内存的问题，本质上只是硬件渲染部分占用的内存，这部分将受到许多因素的影响，包括硬件设备和驱动软件和系统版本等等的影响。这也就是为什么我坚持请你使用 SoftwareOnly 选项的原因。另外，一个应用程序占用的内存是比较复杂的，从专业一点的角度，你不能仅依靠任务管理器里面给你提示的内存来进行了解你的应用进程占用的内存大小。更合适的是，你应该使用类似 vmmap 等专业工具进行测量。一个应用程序占用的内存分为多个部分和多个维度，从共享和独占的角度讲包括共享部分的内存以及 private 部分的内存，从另外的维度讲，应用程序的内存可部分在磁盘里面的虚拟内存和在内存条里面的物理内存里面。对于你提及的 Windows Forms 部分占用比较少的内存，这是因为还有部分在 System 里面的内存没有被你统计到，给了你一些错觉。但是我十分支持你采用你更熟悉的技术完成你的任务
```

```
请帮我将以下内容转述为地道的计算机英文：

这里的行为和 WinUI 3 是相同的，即使在非百分之一百的 DPI 下也是相同的行为
```

```
请帮我将以下内容转述为地道的计算机英文：

在此变更之前，Pressure、Touch Major、Touch Minor在设备变更的时候不会刷新

在此变更之后，可以让 Pressure、Touch Major、Touch Minor 跟随设备变更而刷新属性值
```

```
我想要写信报告一个问题，请你帮我拟一个报告问题的英文标题和一个报告问题的英文内容，要求标题简略，你可以重新组织内容以让内容更加通顺。

以下是我想要报告的问题：

WPF 应用程序在 Intel 设备上可能存在的高内存占用或者内存泄露问题

此问题是在 WPF 仓库的 issues 报告的，经过了漫长时间的调查，我没有什么收获。同时我也感受到了我的能力上的不足，恳请微软能够为此问题投入更多的关注。问题地址是： https://github.com/dotnet/wpf/issues/7704

经过我的初步调查（我的结论可能是错误的），这个问题和 D3D9On12 有比较大的关联关系，即在 D3D9On12 模块下，将原先统计在 GPU 占用的内存部分统计到了进程里面。或者是在 D3D9On12 里面确实会申请或占用更多的内存

此问题一开始是 Edi Wang 报告的，随后经过我的调查，但这个过程中我两都没有什么收获和进度。但随着时间的推移，越来越多的伙伴报告了更多的问题，特别是如 [Link1] 的 ShannonZ 伙伴报告的更加严重的问题

我追踪了很多个 Intel 驱动版本，但从 31.0.101.4032 版本到 32.0.101.5768 版本都没有对此版本有帮助


"Edi Wang"<Edi.Wang@outlook.com>

High memory consumption or memory leak on Intel integrated graphics

https://github.com/dotnet/wpf/issues/7704#issuecomment-2311574479
```


```
你是一个专业英文文档撰写员，请根据以下中文文档，撰写计算机专业的英文文档

[Content Start]

[Content End]
```

```
你是一个专业英文文档撰写员，请根据以下中文内容，撰写计算机专业的英文 Show and tell 文档。你可以修改和添加内容使文档更加流利，要求语气激动，内容富有激情

[Content Start]

标题： 介绍 UISpy.Uno 这款强力的调试 UNO UI 界面的工具的 1.0.0-alpha03 版本

内容：随着 UNO 的 5.2 版本发布，现在 UNO 支持了多窗口功能。UISpy.Uno 这款强力的调试 UNO UI 界面的工具也紧跟着 UNO 的节奏，通过 UNO 的多窗口功能，在相同进程的另一个窗口开启了 UI 调试界面

你可以通过 UISpy.Uno 了解到界面布局信息，界面元素的各个属性信息，甚至可以修改界面元素属性

使用 UISpy.Uno 的方法非常简单，只需要安装 dotnetCampus.UISpy.Uno 库，然后在添加如下的少许代码到你的代码文件里面即可

[Code1]

在你的应用运行起来之后，你可以在你的应用所附加的元素获取键盘焦点时，按下 F12 键开启调试窗口。开启之后的调试界面示例图如下

[Image1]

此 UISpy.Uno 在 GitHub 上使用最好有的 MIT 协议进行开源，欢迎你的使用和参与开发，详细请看：[Link1]

上一次的介绍链接：[Link2]

[Content End]

Title: 
```


## 写通知

```csharp
请帮我写一篇通知，主题是《强化对未成年人使用的智能设备信息内容管理，全面清理违法不良信息》
```

## 代码命名

```csharp
我正在编写一个功能，这个功能是表示转发消息。我正在编写的是 C# 代码，请为这个功能起一个属性名，要求使用 D 字符开头的单词，请给出多个候选命名给我选择
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