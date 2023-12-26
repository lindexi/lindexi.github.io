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


## 翻译

```csharp
请帮我将以下内容进行总结然后翻译成英文：

我统计了我的应用，根据我从 3016336 位用户过去 7 天的数据里面，可以看到平均的启动耗时是 2.7 秒。这里需要进一步说明的是我的应用是一个复杂的应用，启动过程是相当复杂的
```

```
我正在 github 上报告一个问题，请根据以下内容帮我拟定一个英文标题，要求标题简略：

[Content Start]

一个嵌入了 UWP 控件的 WPF 应用程序可能会在 AutomationInteropProvider.HostProviderFromHandle 里卡住

[Content End]

英文标题：
```

```
我正在 github 上回复一个问题，请根据以下内容帮我编写符合程序员风格的英文内容：

[Content Start]
我担心在 #7417 的修复没有彻底修复此问题，原因是我发现我 Cherry-Pick #7417 的更改到我的代码分支时，没有任何的改变，请参阅 https://github.com/dotnet-campus/dotnetCampus.CustomWpf/pull/27 这就意味着我的代码已经合入了修复代码了。然而我还是收到此异常信息

通过阅读代码，我发现 PopupControlService.CloseToolTip 方法的 ToolTip 参数是一个 `_currentToolTip` 字段过来的，这就意味着确实可能在 MouseDevice.Capture 过程中，获取到的 Owner 是空值

[Content End]

英文内容：
```

```
我正在 github 上新建一个 PR 请根据以下内容帮我编写符合程序员风格的英文标题和内容，要求标题简略：

[Content Start]
在 UNO 的 MVU 的生成的绑定代码里面，即在 BindableXxx.cs 里面所生成的代码里面，将会为 XxxModel 层生成对应的 ICommand 命令绑定属性，比如在 XxxModel 的以下方法

    public void Foo(out int n)
    {
        n = 10;
    }

将会在 BindableXxx.cs 里面生成对应的 `public global::Uno.Extensions.Reactive.IAsyncCommand Foo { get; private set; }` 属性

然而在生成 ICommand 命令绑定时，没有为 `out` 参数添加 out 关键字，如上述代码，当前代码生成内容如下

              var n = reactive_arguments;

              model.Foo(n);

以上生成代码将不符合 C# 语法，将导致构建不通过，错误代号是 CS1620

最简的复现 demo 放在： https://github.com/lindexi/lindexi_gd/tree/eeeb023df2f7ab638bafc29d56e1e62a3445cd29/UnoKearqeljikay

[Content End]

英文标题：
```

```
请帮我将以下内容转述为地道的英文：

我担心大量的 DynamicResource 在这个样式里面将会拖慢性能，请问这里能否设计为使用 StaticResource 代替
```

```
我想要写信报告一个问题，请你帮我拟一个报告问题的英文标题和一个报告问题的英文内容，要求标题简略，你可以重新组织内容以让内容更加通顺。

以下是我想要报告的问题：

我观察到最近的 WPF 仓库的关于 Win11 Theming 的更改，在这些更改里面，咱大量使用了 DynamicResource 。咱使用 DynamicResource 的目的如 [Harshit](https://github.com/harshit7962) 在 Xx8579 里面所述，是为了在主题变更的时候，跟随变更界面内容。然而咱都知道，使用 DynamicResource 有着巨大的性能代价，特别是在整个主题样式充满 DynamicResource 的时候，那将会让整个应用的性能都被拖慢。更何况，大量情况下，用户也不会频繁修改系统主题样式，这就意味着通过 DynamicResource 将会拖慢大量用户的性能且不会带来收益。另外，我观察到 [Pankaj Chaurasia](https://github.com/pchaurasia14) 在 Xx8575 里面引入了 ApplicationThemeManager 机制。我认为，当前的 WPF 缺乏了明确的跟随主题变更的机制，换句话说我认为依靠 DynamicResource 来实现跟随主题变更是一个较差的实现方式。我建议在 WPF 仓库里面同步引入一套新的机制，可以用来代替现有的 DynamicResource 代码，用来正确且高效的实现让样式跟随主题的变更
```

## 写通知

```csharp
请帮我写一篇通知，主题是《强化对未成年人使用的智能设备信息内容管理，全面清理违法不良信息》
```

## 代码命名

```csharp
我正在编写一个功能，这个功能是表示连笔功能。我正在编写的是 C# 代码，请为这个功能起一个类型名，请给出多个候选命名给我选择
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