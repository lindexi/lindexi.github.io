---
title: dotnet 读 WPF 源代码笔记 聊聊 OpenType 定义的字体特性
description: 本文记录我读 WPF 源代码的 OpenType 字体特性标签 OpenType Feature Tags 的笔记内容
tags: WPF,dotnet
category: 
---

<!-- CreateTime:2026/03/24 08:49:45 -->

<!-- 发布 -->
<!-- 博客 -->

本文部分内容由 AI 辅助记录和整理

在 WPF 里面，关于 Feature Tag 字体特性的定义是放在 `src\Microsoft.DotNet.Wpf\src\PresentationCore\MS\Internal\FontFace\Tags.cs` 文件里面，定义的枚举代码内容如下

```csharp
    internal enum FeatureTags
    {
        AccessAllAlternates                      = 0x61616c74, // 'aalt'
        AboveBaseForms                           = 0x61627666, // 'abvf'
        AboveBaseMarkPositioning                 = 0x6162766d, // 'abvm'
        AboveBaseSubstitutions                   = 0x61627673, // 'abvs'
        AlternativeFractions                     = 0x61667263, // 'afrc'
        Akhands                                  = 0x616b686e, // 'akhn'
        BelowBaseForms                           = 0x626c7766, // 'blwf'
        BelowBaseMarkPositioning                 = 0x626c776d, // 'blwm'
        BelowBaseSubstitutions                   = 0x626c7773, // 'blws'
        PetiteCapitalsFromCapitals               = 0x63327063, // 'c2pc'
        SmallCapitalsFromCapitals                = 0x63327363, // 'c2sc'
        ContextualAlternates                     = 0x63616c74, // 'calt'
        CaseSensitiveForms                       = 0x63617365, // 'case'
        GlyphCompositionDecomposition            = 0x63636d70, // 'ccmp'
        Conjunctformafterro                      = 0x63666172, // 'cfar'
        ContextualLigatures                      = 0x636c6967, // 'clig'
        Conjuncts                                = 0x636a6374, // 'cjct'
        CapitalSpacing                           = 0x63707370, // 'cpsp'
        ContextualSwash                          = 0x63737768, // 'cswh'
        CursivePositioning                       = 0x63757273, // 'curs'
        DefaultProcessing                        = 0x64666c74, // 'dflt'
        Distances                                = 0x64697374, // 'dist'
        DiscretionaryLigatures                   = 0x646c6967, // 'dlig'
        Denominators                             = 0x646e6f6d, // 'dnom'
        Diphthongs                               = 0x64706e67, // 'dpng'
        ExpertForms                              = 0x65787074, // 'expt'
        FinalglyphAlternates                     = 0x66616c74, // 'falt'
        TerminalForms                            = 0x66696e61, // 'fina'
        TerminalForms2                           = 0x66696e32, // 'fin2'
        TerminalForms3                           = 0x66696e33, // 'fin3'
        Fractions                                = 0x66726163, // 'frac'
        FullWidth                                = 0x66776964, // 'fwid'
        HalfForms                                = 0x68616c66, // 'half'
        HalantForms                              = 0x68616c6e, // 'haln'
        AlternateHalfWidth                       = 0x68616c74, // 'halt'
        HistoricalForms                          = 0x68697374, // 'hist'
        HorizontalKanaAlternates                 = 0x686b6e61, // 'hkna'
        HistoricalLigatures                      = 0x686c6967, // 'hlig'
        Hangul                                   = 0x686e676c, // 'hngl'
        HalfWidth                                = 0x68776964, // 'hwid'
        HojoKanjiForms                           = 0x686f6a6f, // 'hojo'
        InitialForms                             = 0x696e6974, // 'init'
        IsolatedForms                            = 0x69736f6c, // 'isol'
        Italics                                  = 0x6974616c, // 'ital'
        JapaneseForms                            = 0x6a616a70, // 'jajp'
        JustificationAlternatives                = 0x6a616c74, // 'jalt'
        JIS04Forms                               = 0x6a703034, // 'jp04'
        JIS78Forms                               = 0x6a703738, // 'jp78'
        JIS83Forms                               = 0x6a703833, // 'jp83'
        JIS90Forms                               = 0x6a703930, // 'jp90'
        Kerning                                  = 0x6b65726e, // 'kern'
        LeftBounds                               = 0x6c666264, // 'lfbd'
        StandardLigatures                        = 0x6c696761, // 'liga'
        LeadingJamoForms                         = 0x6c6a6d6f, // 'ljmo'
        LiningFigures                            = 0x6c6e756d, // 'lnum'
        LocalizedForms                           = 0x6c6f636c, // 'locl'
        MarkPositioning                          = 0x6d61726b, // 'mark'
        MedialForms                              = 0x6d656469, // 'medi'
        MedialForms2                             = 0x6d656432, // 'med2'
        MathematicalGreek                        = 0x6d67726b, // 'mgrk'
        MarktoMarkPositioning                    = 0x6d6b6d6b, // 'mkmk'
        MarkPositioningviaSubstitution           = 0x6d736574, // 'mset'
        AlternateAnnotationForms                 = 0x6e616c74, // 'nalt'
        NLCKanjiForms                            = 0x6e6c636b, // 'nlck'
        NuktaForms                               = 0x6e756b74, // 'nukt'
        Numerators                               = 0x6e756d72, // 'numr'
        OldStyleFigures                          = 0x6f6e756d, // 'onum'
        OpticalBounds                            = 0x6f706264, // 'opbd'
        Ordinals                                 = 0x6f72646e, // 'ordn'
        Ornaments                                = 0x6f726e6d, // 'ornm'
        ProportionalAlternateWidth               = 0x70616c74, // 'palt'
        PetiteCapitals                           = 0x70636170, // 'pcap'
        ProportionalFigures                      = 0x706e756d, // 'pnum'
        PrebaseForms                             = 0x70726566, // 'pref'
        PrebaseSubstitutions                     = 0x70726573, // 'pres'
        PostbaseForms                            = 0x70737466, // 'pstf'
        PostbaseSubstitutions                    = 0x70737473, // 'psts'
        ProportionalWidths                       = 0x70776964, // 'pwid'
        QuarterWidths                            = 0x71776964, // 'qwid'
        Randomize                                = 0x72616e64, // 'rand'
        RakarForms                               = 0x726b7266, // 'rkrf'
        RequiredLigatures                        = 0x726c6967, // 'rlig'
        RephForm                                 = 0x72706866, // 'rphf'
        RightBounds                              = 0x72746264, // 'rtbd'
        RightToLeftAlternates                    = 0x72746c61, // 'rtla'
        RubyNotationForms                        = 0x72756279, // 'ruby'
        StylisticAlternates                      = 0x73616c74, // 'salt'
        ScientificInferiors                      = 0x73696e66, // 'sinf'
        OpticalSize                              = 0x73697a65, // 'size'
        SmallCapitals                            = 0x736d6370, // 'smcp'
        SimplifiedForms                          = 0x736d706c, // 'smpl'
        StylisticSet1                            = 0x73733031, // 'ss01'
        StylisticSet2                            = 0x73733032, // 'ss02'
        StylisticSet3                            = 0x73733033, // 'ss03'
        StylisticSet4                            = 0x73733034, // 'ss04'
        StylisticSet5                            = 0x73733035, // 'ss05'
        StylisticSet6                            = 0x73733036, // 'ss06'
        StylisticSet7                            = 0x73733037, // 'ss07'
        StylisticSet8                            = 0x73733038, // 'ss08'
        StylisticSet9                            = 0x73733039, // 'ss09'
        StylisticSet10                           = 0x73733130, // 'ss10'
        StylisticSet11                           = 0x73733131, // 'ss11'
        StylisticSet12                           = 0x73733132, // 'ss12'
        StylisticSet13                           = 0x73733133, // 'ss13'
        StylisticSet14                           = 0x73733134, // 'ss14'
        StylisticSet15                           = 0x73733135, // 'ss15'
        StylisticSet16                           = 0x73733136, // 'ss16'
        StylisticSet17                           = 0x73733137, // 'ss17'
        StylisticSet18                           = 0x73733138, // 'ss18'
        StylisticSet19                           = 0x73733139, // 'ss19'
        StylisticSet20                           = 0x73733230, // 'ss20'
        Subscript                                = 0x73756273, // 'subs'
        Superscript                              = 0x73757073, // 'sups'
        Swash                                    = 0x73777368, // 'swsh'
        Titling                                  = 0x7469746c, // 'titl'
        TrailingJamoForms                        = 0x746a6d6f, // 'tjmo'
        TraditionalNameForms                     = 0x746e616d, // 'tnam'
        TabularFigures                           = 0x746e756d, // 'tnum'
        TraditionalForms                         = 0x74726164, // 'trad'
        ThirdWidths                              = 0x74776964, // 'twid'
        Unicase                                  = 0x756e6963, // 'unic'
        AlternateVerticalMetrics                 = 0x76616c74, // 'valt'
        VattuVariants                            = 0x76617475, // 'vatu'
        VerticalWriting                          = 0x76657274, // 'vert'
        AlternateVerticalHalfMetrics             = 0x7668616c, // 'vhal'
        VowelJamoForms                           = 0x766a6d6f, // 'vjmo'
        VerticalKanaAlternates                   = 0x766b6e61, // 'vkna'
        VerticalKerning                          = 0x766b726e, // 'vkrn'
        ProportionalAlternateVerticalMetrics     = 0x7670616c, // 'vpal'
        VerticalRotation                         = 0x76727432, // 'vrt2'
        SlashedZero                              = 0x7a65726f, // 'zero'
    }
```

这个枚举对应的就是 OpenType 规范里定义的字体特性，每个特性都是 4 字符的标签转的整数。在 WPF 代码里面，直接给出的是整数，但在注释里面写明了对应的字符

整个 OpenType 规范里定义的字体特性可以被分为几个大类，比如通用排版、数字相关、大小写相关、连字相关、东亚文字（中日朝、印度文字相关）、特殊字形变体、定位相关这些

首先第一大类：【通用字形变换&定位类】

- AccessAllAlternates（aalt）：访问所有替代字形，开启后可以调用同一字符对应的所有设计变体字形，一般字体设计工具、排版软件里选特殊字形的时候用。
- GlyphCompositionDecomposition（ccmp）：字形组合/分解，是默认开启的基础特性，处理字符的合字拆分、比如把带重音的字符拆成基础字符+重音符号，或者反过来组合，保证复杂字符正确渲染。
- ContextualAlternates（calt）：上下文替代字形，根据相邻字符自动替换字形，比如手写体里同一个字母在不同位置写法不一样，就是这个特性控制，默认大部分排版场景开启。
- Randomize（rand）：随机替换字形，给同一个字符随机选不同的变体，模拟手写的自然感，适合手写字体、艺术排版。
- StylisticAlternates（salt）：风格替代字形，提供整组的风格化字形替换，比单独替换更统一，比如字体的圆润风格、尖锐风格整组切换。
- StylisticSet1~20（ss01~ss20）：风格集1-20，字体设计师可以自定义最多20组不同的风格预设，每个ssxx对应一组独立的字形变体，用户可以按需切换，比如有的字体ss01是无衬线变体，ss02是手写变体之类的。
- Swash（swsh）：花体字，替换为带装饰性的花体字形，一般用于标题、海报设计的西文字体。
- ContextualSwash（cswh）：上下文花体字，根据字符位置自动调整花体的样式，避免和相邻字符重叠，更自然。
- Titling（titl）：标题字形，专门为大字号标题设计的字形，通常更简洁、笔画更粗，避免小字号用的细节在大字号下显得冗余。
- LocalizedForms（locl）：本地化字形，根据语言区域自动切换对应字形，比如同一个汉字在中国大陆、台湾、日本的字形不一样，拉丁文在不同欧洲国家的变体，都靠这个特性控制。
- HistoricalForms（hist）：历史字形，切换为文字的古老历史写法，比如古英文、繁体古字，适合古籍排版、历史题材设计。
- Ornaments（ornm）：装饰字形，调用字体里附带的装饰符号、花纹、边框等图形字形。
- CursivePositioning（curs）：草书/手写体连接定位，处理手写体字符之间的自然连接，保证每个字的收尾和下一个字的起笔完美衔接。
- DefaultProcessing（dflt）：默认处理，是字体的基础默认特性集，一般渲染引擎默认加载。
- Unicase（unic）：大小写混排，把大写和小写字母统一成同一种高度的混合字形，适合特殊风格的排版。
- OpticalSize（size）：光学尺寸适配，根据字号自动调整字形的笔画粗细、间距，小字号下更清晰，大字号下更美观。

第二大类：【间距&定位类】

- Kerning（kern）：这是字间距/字距调整，是最常用的特性，调整特定字符对之间的间距，比如「W」和「a」之间的间距会比普通字符对小，避免显得空隙太大，默认排版都开启。
- VerticalKerning（vkrn）：竖排字距调整，对应竖排文本的字距优化。
- CapitalSpacing（cpsp）：大写字母间距，专门调整全大写文本的字符间距，让大写文本更透气易读。
- MarkPositioning（mark）：标记定位，调整重音符号、注音符号等标记相对于基础字符的位置，保证标记不偏移、不重叠。
- MarktoMarkPositioning（mkmk）：标记到标记定位，调整多个叠加的标记之间的位置，比如一个字符上同时加两个重音的场景，常见于越南文、印度系文字。
- MarkPositioningviaSubstitution（mset）：通过替换实现标记定位，部分旧字体用替换的方式处理标记位置，兼容老设备。
- OpticalBounds（opbd）：视觉边界调整，根据字形的实际视觉边缘调整对齐位置，而不是用字体的 bounding box 对齐，让排版更整齐，比如大写字母「T」的顶部会稍微突出一点，opbd会调整让视觉上对齐。
- LeftBounds（lfbd）：左边界调整，优化字符左侧的对齐。
- RightBounds（rtbd）：右边界调整，优化字符右侧的对齐。
- Distances（dist）：距离调整，处理复杂文字系统里的字符间距，常见于印度系文字。
- JustificationAlternatives（jalt）：对齐替代字形，两端对齐的时候自动调整部分字符的宽度，避免出现太大的空格，让对齐更自然。

第三大类：【连字（Ligature）类】

- StandardLigatures（liga）：标准连字，默认开启的常用连字，比如把「fi」「fl」合并成一个连字，避免「f」的一横和「i」的点重叠，提升易读性。
- ContextualLigatures（clig）：上下文连字，根据相邻字符自动生成连字，比标准连字更灵活，常见于手写体、阿拉伯文。
- DiscretionaryLigatures（dlig）： discretionary是 discretionary的？哦 discretionary是可选的、 discretionary连字，就是装饰性连字，比如「ff」「ffi」「st」这类艺术化的连字，默认不开，需要做艺术排版的时候开。
- HistoricalLigatures（hlig）：历史连字，比如古英文的「æ」「œ」这类古老的连字写法，适合古籍排版。
- RequiredLigatures（rlig）：必需连字，某些文字系统必须的连字，比如阿拉伯文、希伯来文，如果不开这些连字文字根本无法正确显示，渲染引擎默认强制开启。

第四大类：【数字相关类】

- LiningFigures（lnum）： lining数字，就是大写等高数字，和大写字母高度一致，适合和大写文字混排。
- OldStyleFigures（onum）：旧风格数字，也叫小写数字，有高低错落，和小写字母混排更协调，适合正文排版。
- ProportionalFigures（pnum）：比例宽度数字，每个数字宽度根据本身的形状调整，比如「1」比「8」窄，适合正文，更美观。
- TabularFigures（tnum）：表格宽度数字，所有数字宽度一致，对齐整齐，专门用于表格、财务数据、价格显示，保证上下行对齐。
- Numerators（numr）：分子数字，显示分数的分子形态，比普通数字小，位置靠上。
- Denominators（dnom）：分母数字，显示分数的分母形态，比普通数字小，位置靠下。
- Fractions（frac）：分数渲染，自动把「1/2」这类格式转换成标准的分数字形，不用手动调整大小位置。
- AlternativeFractions（afrc）：替代分数，支持对角线样式的分数，或者更复杂的分数写法。
- Subscript（subs）：下标字形，自动把字符切换为下标形态，比如化学公式H₂O的2。
- Superscript（sups）：上标字形，自动切换为上标形态，比如平方米m²的2，或者注脚标记。
- ScientificInferiors（sinf）：科学下标，专门用于科学公式的下标，比普通下标更规范，适合学术排版。
- Ordinals（ordn）：序数词后缀，比如英文的1st、2nd里的st、nd自动变成上标形态，更规范。
- SlashedZero（zero）：带斜杠的零，把数字0显示为带斜杠的样式，避免和字母O混淆，适合代码、财务、ID编号的显示场景，非常实用。

第五大类：【大小写相关类】

- SmallCapitals（smcp）：小型大写字母，把小写字母转换成和x高度一致的小型大写，比全大写更柔和，适合正文里的缩写，比如「NASA」用小型大写显示不会太突兀。
- PetiteCapitals（pcap）：超小型大写字母，比小型大写更小，适合更精细的排版需求。
- SmallCapitalsFromCapitals（c2sc）：从大写转小型大写，把输入的大写字母直接转换成小型大写，不用输入小写再转。
- PetiteCapitalsFromCapitals（c2pc）：从大写转超小型大写，同上。
- CaseSensitiveForms（case）：大小写敏感字形，全大写排版的时候自动调整标点、符号的位置，让和大写字母对齐更协调，比如括号、引号的位置会往上移。
- Italics（ital）：意大利斜体，自动切换为斜体字形，比单纯的算法倾斜更美观，笔画是专门设计的。

第六大类：【东亚文字（中日朝）相关类】

- FullWidth（fwid）：全宽字符，把半宽字符转换成全宽，和中文等宽对齐，适合中日韩混排。
- HalfWidth（hwid）：半宽字符，把全宽字符转换成半宽，节省空间。
- QuarterWidths（qwid）：1/4宽字符，东亚文字的窄宽度变体，适合排版紧凑的场景。
- ThirdWidths（twid）：1/3宽字符，同上，宽度介于半宽和1/4宽之间。
- ProportionalWidths（pwid）： proportional宽度东亚字符，每个字符宽度根据形状调整，不是统一宽度，更美观。
- AlternateHalfWidth（halt）：替代半宽，半宽字符的变体，对齐更友好。
- ProportionalAlternateWidth（palt）： proportional替代宽度，东亚字符的比例宽度变体，排版更自然。
- RubyNotationForms（ruby）： ruby注音字形，专门为注音（比如日文振假名、中文拼音标注）设计的小字号字形，小字号下更清晰。
- HorizontalKanaAlternates（hkna）：横排假名变体，横排的时候用的假名优化字形，更易读。
- VerticalKanaAlternates（vkna）：竖排假名变体，竖排的时候用的假名优化字形，符合竖排阅读习惯。
- VerticalWriting（vert）：竖排书写，把字符切换为竖排专用字形，比如竖排的标点、汉字会调整方向和形状，适合中日韩竖排文本。
- VerticalRotation（vrt2）：竖排旋转，把横排字符旋转90度适配竖排，比如英文、数字在竖排文本里自动旋转。
- AlternateVerticalMetrics（valt）：替代垂直度量，竖排的时候调整字符的垂直间距。
- AlternateVerticalHalfMetrics（vhal）：替代半高垂直度量，竖排半高字符的间距调整。
- ProportionalAlternateVerticalMetrics（vpal）：比例垂直度量，竖排比例宽度字符的间距调整。
- SimplifiedForms（smpl）：简体字形，切换为简体中文字形。
- TraditionalForms（trad）：繁体字形，切换为繁体中文字形。
- JIS78Forms（jp78）：JIS 1978标准的日文字形。
- JIS83Forms（jp83）：JIS 1983标准的日文字形。
- JIS90Forms（jp90）：JIS 1990标准的日文字形。
- JIS04Forms（jp04）：JIS 2004标准的日文字形。
- HojoKanjiForms（hojo）：日本法务省户籍用汉字字形。
- NLCKanjiForms（nlck）：日本国立国语研究所的汉字字形标准。
- JapaneseForms（jajp）：日本本地化字形。
- TraditionalNameForms（tnam）：传统人名用字形，比如中文、日文的人名异体字，排版人名的时候自动切换为正确的写法。
- Hangul（hngl）：朝鲜语 Hangul 谚文字形优化。
- LeadingJamoForms（ljmo）：谚文初声（首字母）变体。
- VowelJamoForms（vjmo）：谚文中声（元音）变体。
- TrailingJamoForms（tjmo）：谚文终声（尾音）变体，这三个都是处理谚文组合的基础特性，保证谚文正确合成。

第七大类：【印度系/南亚文字相关类】（这些都是天城文、泰米尔文等南亚文字的专用特性，普通中文排版很少用到）

- AboveBaseForms（abvf）：基线以上字形
- AboveBaseMarkPositioning（abvm）：基线以上标记定位
- AboveBaseSubstitutions（abvs）：基线以上替换
- Akhands（akhn）：天城文连字
- BelowBaseForms（blwf）：基线以下字形
- BelowBaseMarkPositioning（blwm）：基线以下标记定位
- BelowBaseSubstitutions（blws）：基线以下替换
- Conjunctformafterro（cfar）：Ro后连字形式
- Conjuncts（cjct）：辅音连字
- Diphthongs（dpng）：双元音变体
- ExpertForms（expt）：专家用印度文字形
- FinalglyphAlternates（falt）：末尾字形变体
- TerminalForms（fina/fin2/fin3）：词尾变体，不同的词尾形式
- HalfForms（half）：半形辅音
- HalantForms（haln）：Halant标记变体
- InitialForms（init）：词首变体
- IsolatedForms（isol）：独立字形（单词单独出现的时候用的形式，比如阿拉伯文、印度文每个字符有词首/词中/词尾/独立四种形式）
- MedialForms（medi/med2）：词中变体
- NuktaForms（nukt）：Nukta标记变体
- PrebaseForms（pref）：基线前字形
- PrebaseSubstitutions（pres）：基线前替换
- PostbaseForms（pstf）：基线后字形
- PostbaseSubstitutions（psts）：基线后替换
- RakarForms（rkrf）：Rakar标记变体
- RephForm（rphf）：Reph标记变体
- VattuVariants（vatu）：Vattu标记变体
- RightToLeftAlternates（rtla）：从右到左文字的变体，适配阿拉伯文、希伯来文等RTL文字。

最后需要说明的是，这些特性大部分都需要字体本身支持才能生效，不是开启了就有用，如果字体没有对应特性的设计，开启了也不会有变化

以下为 AI 整理的表格内容

## 一、通用排版核心类（最常用）

| 枚举名 | 标签 | 含义&用途 |
|--------|------|-----------|
| Kerning | kern | 字距调整，默认开启，优化特定字符对（比如W和a、T和i）的间距，避免空隙过大或重叠 |
| ContextualAlternates | calt | 上下文替代字形，默认开启，根据相邻字符自动替换字形（比如手写体中同一个字母在词首/词中写法不同） |
| StandardLigatures | liga | 标准连字，默认开启，把fi、fl这类容易重叠的字符组合合并成专门设计的连字，提升易读性 |
| GlyphCompositionDecomposition | ccmp | 字形组合/分解，基础特性，处理带重音字符、复杂字符的拆分合并，保证文字正确渲染 |
| LocalizedForms | locl | 本地化字形，根据语言区域自动切换对应字形，比如同一个汉字在大陆/台湾/日本的不同写法、拉丁文的区域变体 |
| CaseSensitiveForms | case | 大小写敏感适配，全大写排版时自动调整标点、符号的位置，和大写字母对齐更协调 |
| Randomize | rand | 随机字形替换，给同一个字符随机选不同变体，模拟手写的自然错落感，适合艺术字体、手写场景 |
| StylisticAlternates | salt | 风格替代整组切换，调用字体设计师预设的整套风格化字形，比如统一切换为圆润/尖锐风格 |
| StylisticSet1~20 | ss01~ss20 | 风格集1-20，字体最多支持20组自定义风格预设，每个集对应一套独立的字形变体，按需切换 |
| Swash | swsh | 花体字，替换为带装饰性的花体字形，多用于西文标题、海报设计 |
| OpticalSize | size | 光学尺寸适配，根据字号自动调整笔画粗细、间距，小字号更清晰，大字号更美观 |



## 二、数字相关类

| 枚举名 | 标签 | 含义&用途 |
|--------|------|-----------|
| LiningFigures | lnum | 等高数字，和大写字母高度一致，适合和大写文字混排 |
| OldStyleFigures | onum | 旧风格/小写数字，有高低错落，和小写字母混排更协调，适合正文排版 |
| ProportionalFigures | pnum | 比例宽度数字，每个数字宽度按形状调整（比如1比8窄），美观性好，适合正文 |
| TabularFigures | tnum | 表格等宽数字，所有数字宽度完全一致，上下对齐整齐，专门用于表格、财务数据、价格显示 |
| Fractions | frac | 自动分数渲染，把1/2这类格式直接转成标准分数字形，无需手动调整大小位置 |
| Subscript | subs | 自动下标，比如化学公式H₂O的2，无需手动调整字号位置 |
| Superscript | sups | 自动上标，比如平方米m²的2、注脚标记 |
| Ordinals | ordn | 序数词后缀适配，比如英文1st、2nd里的st/nd自动转为上标形态 |
| SlashedZero | zero | 带斜杠的零，把0显示为带斜杠的样式，避免和字母O混淆，非常适合代码、ID编号、财务场景 |
| Numerators/Denominators | numr/dnom | 分子/分母专属字形，用于复杂分数显示 |


## 三、大小写和连字类

| 枚举名 | 标签 | 含义&用途 |
|--------|------|-----------|
| SmallCapitals | smcp | 小型大写字母，把小写字母转成和小写x高度一致的小型大写，比全大写更柔和，适合正文里的缩写 |
| PetiteCapitals | pcap | 超小型大写字母，比小型大写更小，适合精细排版 |
| DiscretionaryLigatures | dlig | 装饰性连字，默认关闭，启用后显示ff、ffi、st这类艺术化连字，适合设计场景 |
| HistoricalLigatures | hlig | 历史连字，显示古英文、古籍里的古老连字（比如æ、œ） |
| RequiredLigatures | rlig | 必需连字，阿拉伯文、希伯来文等文字必须的连字，不开启文字就无法正确显示，渲染引擎默认强制开启 |
| Italics | ital | 原生斜体，切换为字体专门设计的斜体字形，比算法倾斜更美观 |


## 四、东亚文字（中日朝）专用类

| 枚举名 | 标签 | 含义&用途 |
|--------|------|-----------|
| FullWidth/HalfWidth | fwid/hwid | 全宽/半宽切换，中日韩混排时统一字符宽度，对齐更整齐 |
| VerticalWriting | vert | 竖排适配，切换为竖排专用字形，调整标点、汉字的方向和形状，符合中日韩竖排阅读习惯 |
| VerticalRotation | vrt2 | 竖排旋转，把横排的英文、数字自动旋转90度适配竖排文本 |
| RubyNotationForms | ruby | 注音专用字形，专门为日文振假名、中文拼音标注等小字号场景设计，小字号下更清晰 |
| SimplifiedForms/TraditionalForms | smpl/trad | 简体/繁体字形切换 |
| JIS78/JIS83/JIS90/JIS04 | jp78~jp04 | 不同年代JIS标准的日文字形切换 |
| HojoKanjiForms | hojo | 日本法务省户籍用汉字字形 |
| Hangul/LeadingJamoForms/VowelJamoForms/TrailingJamoForms | hngl/ljmo/vjmo/tjmo | 谚文（朝鲜语）合成适配，保证谚文的首/中/尾音正确组合显示 |


## 五、南亚/RTL文字专用类（普通中文排版很少用到）

这类都是天城文、泰米尔文、阿拉伯文等复杂文字系统的专属特性，用于处理字符的基线上下位置、词首/词中/词尾变体、连字合成：
`AboveBaseForms/AboveBaseMarkPositioning/AboveBaseSubstitutions`、`Akhands`、`BelowBaseForms/BelowBaseMarkPositioning/BelowBaseSubstitutions`、`Conjuncts`、`InitialForms/IsolatedForms/MedialForms/TerminalForms`、`RakarForms/RephForm/NuktaForms`、`RightToLeftAlternates`等，都是保证复杂文字正确渲染的基础特性。

以上就是我读 WPF 源代码的关于字体特性标签 OpenType Feature Tags 的笔记内容

当前的 WPF 在 [https://github.com/dotnet/wpf](https://github.com/dotnet/wpf) 完全开源，使用友好的 MIT 协议，意味着允许任何人任何组织和企业任意处置，包括使用，复制，修改，合并，发表，分发，再授权，或者销售。在仓库里面包含了完全的构建逻辑，只需要本地的网络足够好（因为需要下载一堆构建工具），即可进行本地构建
