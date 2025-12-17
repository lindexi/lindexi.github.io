---
title: DirectWrite 通过 GetUnicodeRanges 获取字体能支持的字符范围
description: 一个字体文件只能支持有限的字符数量，为了能够知道某个字体包含哪些字符，可通过 DirectWrite 提供的 GetUnicodeRanges 方法获取。本文将演示如何从 IDWriteFontFace 的 GetUnicodeRanges 方法获取字体能支持的字符范围以及对比 WPF 的行为
tags: 
category: 
---

<!-- CreateTime:2025/12/17 07:01:08 -->

<!-- 发布 -->
<!-- 博客 -->

需要通过以下路径才能获取到 IDWriteFontFace3 实例

- 通过 DWrite 的 DWriteCreateFactory 创建 IDWriteFactory 工厂
- 通过 IDWriteFactory 工厂的 CreateFontFaceReference 方法，从字体文件获取到 IDWriteFontFaceReference 对象
- 从 IDWriteFontFaceReference 的 CreateFontFace 方法获取到 IDWriteFontFace3 实例

拿到 IDWriteFontFace3 实例之后，即可调用 GetUnicodeRanges 方法获取字体能支持的字符范围数组

以下示例代码将尝试获取宋体字体能够支持的字符范围

先创建 IDWriteFactory 工厂，代码如下

```csharp
        DWrite dWrite = DWrite.GetApi();
        ComPtr<IDWriteFactory6> factory = dWrite.DWriteCreateFactory<IDWriteFactory6>(FactoryType.Shared);
```

从字体文件获取 IDWriteFontFaceReference 对象，代码如下

```csharp
    private const string FontFile = @"C:\windows\fonts\simsun.ttc";

        // 宋体字体
        var fontFile = FontFile;

            IDWriteFontFaceReference* fontFaceReference;

            fixed (char* pFontFile = fontFile)
            {
                hr = factory.Handle->CreateFontFaceReference(pFontFile, null, (uint) 0, FontSimulations.None,
                    &fontFaceReference);
                hr.Throw();
            }
```

从 IDWriteFontFaceReference 获取 IDWriteFontFace3 实例，代码如下

```csharp
            IDWriteFontFace3* fontFace3;
            fontFaceReference->CreateFontFace(&fontFace3);
```

调用 IDWriteFontFace3 的 GetUnicodeRanges 获取字符范围时，需要调用 GetUnicodeRanges 两次，第一次调用是获取数组长度，第二次才是获取字符范围，代码如下

```csharp
            uint rangeCount = 0;
            fontFace3->GetUnicodeRanges(0, null, ref rangeCount);
            var unicodeRanges = new UnicodeRange[rangeCount];

            fixed (UnicodeRange* p = unicodeRanges)
            {
                fontFace3->GetUnicodeRanges(rangeCount, p, ref rangeCount);
            }
```

尝试拿到的 UnicodeRange 数组就包含了字体文件能够支持的字符范围了

获取之后，尝试输出到控制台，输出代码如下

```csharp
            fixed (UnicodeRange* p = unicodeRanges)
            {
                fontFace3->GetUnicodeRanges(rangeCount, p, ref rangeCount);
            }

            for (var i = 0; i < unicodeRanges.Length; i++)
            {
                var unicodeRange = unicodeRanges[i];
                var start = new Rune(unicodeRange.First);
                var end = new Rune(unicodeRange.Last);

                Console.WriteLine($"Range {i}: '{start.ToString()}'({start.Value}) - '{end.ToString()}'({end.Value}) Length={end.Value - start.Value + 1}");
            }
```

尝试运行项目，可见控制台的输出大概如下

```csharp
Range 0: ' '(32) - ''(127) Length=96
Range 1: '?'(160) - '?'(255) Length=96
Range 2: 'ā'(257) - 'ā'(257) Length=1
...
Range 130: '?'(13312) - '?'(19903) Length=6592
Range 131: '一'(19968) - '?'(40959) Length=20992
...
Range 156: '！'(65281) - '～'(65374) Length=94
Range 157: '￠'(65504) - '￥'(65509) Length=6
```

相比之下，在 WPF 框架内，获取字体能够支持的字符范围就简单多了。只需用 GlyphTypeface 的 CharacterToGlyphMap 获取即可。对比的测试逻辑如下

```csharp
            uint rangeCount = 0;
            fontFace3->GetUnicodeRanges(0, null, ref rangeCount);
            var unicodeRanges = new UnicodeRange[rangeCount];
            fixed (UnicodeRange* p = unicodeRanges)
            {
                fontFace3->GetUnicodeRanges(rangeCount, p, ref rangeCount);
            }

            TestWpf(unicodeRanges);

    private void TestWpf(IReadOnlyList<UnicodeRange> unicodeRanges)
    {
        var wpfFontFamily = new FontFamily("宋体");
        Typeface typeface = wpfFontFamily.GetTypefaces().First();
        if (typeface.TryGetGlyphTypeface(out var glyphTypeface))
        {
            for (uint i = 0; i < 6000; i++)
            {
                if (IsInUnicodeRange(i))
                {
                    if (glyphTypeface.CharacterToGlyphMap.TryGetValue((int) i, out var glyphIndex))
                    {
                        // 在范围内的字符，可以找到对应的字形索引
                        _ = glyphIndex;
                    }
                    else
                    {
                        // 在范围内的字符，找不到对应的字形索引
                        Console.WriteLine($"Character {i} is not in the glyph map.");
                        Debugger.Break();
                    }
                }
                else
                {
                    // 不在范围内的字符，预期找不到对应的字形索引
                    if (glyphTypeface.CharacterToGlyphMap.TryGetValue((int) i, out var glyphIndex))
                    {
                        // 不在范围内的字符，居然可以找到对应的字形索引
                        _ = glyphIndex;
                        Debugger.Break();
                    }
                    else
                    {
                        // 不在范围内的字符，预期找不到对应的字形索引
                    }
                }
            }

            bool IsInUnicodeRange(uint codepoint)
            {
                foreach (var unicodeRange in unicodeRanges)
                {
                    if (codepoint >= unicodeRange.First && codepoint <= unicodeRange.Last)
                    {
                        return true;
                    }
                }

                return false;
            }
        }
    }
```

尝试运行以上代码，可以看到所有能够从 UnicodeRange 数组里面找到的字符，同样也能从 WPF 的 CharacterToGlyphMap 找到。所有在 UnicodeRange 数组范围之外的，也都不能在 CharacterToGlyphMap 找到。可以认为 WPF 的 CharacterToGlyphMap 的行为就和 GetUnicodeRanges 相同

核心代码如下

```csharp

    // 宋体字体
    private const string FontFile = @"C:\windows\fonts\simsun.ttc";

    private void MainWindow_Loaded(object sender, RoutedEventArgs e)
    {
        DWrite dWrite = DWrite.GetApi();
        ComPtr<IDWriteFactory6> factory = dWrite.DWriteCreateFactory<IDWriteFactory6>(FactoryType.Shared);

        // 宋体字体
        var fontFile = FontFile;

        unsafe
        {
            HResult hr = 0;

            IDWriteFontFaceReference* fontFaceReference;

            fixed (char* pFontFile = fontFile)
            {
                hr = factory.Handle->CreateFontFaceReference(pFontFile, null, (uint) 0, FontSimulations.None,
                    &fontFaceReference);
                hr.Throw();
            }

            IDWriteFontFace3* fontFace3;
            fontFaceReference->CreateFontFace(&fontFace3);

            uint rangeCount = 0;
            fontFace3->GetUnicodeRanges(0, null, ref rangeCount);
            var unicodeRanges = new UnicodeRange[rangeCount];

            fixed (UnicodeRange* p = unicodeRanges)
            {
                fontFace3->GetUnicodeRanges(rangeCount, p, ref rangeCount);
            }

            for (var i = 0; i < unicodeRanges.Length; i++)
            {
                var unicodeRange = unicodeRanges[i];
                var start = new Rune(unicodeRange.First);
                var end = new Rune(unicodeRange.Last);

                Console.WriteLine($"Range {i}: '{start.ToString()}'({start.Value}) - '{end.ToString()}'({end.Value}) Length={end.Value - start.Value + 1}");
            }

            TestWpf(unicodeRanges);
        }
    }

    private void TestWpf(IReadOnlyList<UnicodeRange> unicodeRanges)
    {
        var wpfFontFamily = new FontFamily("宋体");
        Typeface typeface = wpfFontFamily.GetTypefaces().First();
        if (typeface.TryGetGlyphTypeface(out var glyphTypeface))
        {
            for (uint i = 0; i < 6000; i++)
            {
                if (IsInUnicodeRange(i))
                {
                    if (glyphTypeface.CharacterToGlyphMap.TryGetValue((int) i, out var glyphIndex))
                    {
                        // 在范围内的字符，可以找到对应的字形索引
                        _ = glyphIndex;
                    }
                    else
                    {
                        // 在范围内的字符，找不到对应的字形索引
                        Console.WriteLine($"Character {i} is not in the glyph map.");
                        Debugger.Break();
                    }
                }
                else
                {
                    // 不在范围内的字符，预期找不到对应的字形索引
                    if (glyphTypeface.CharacterToGlyphMap.TryGetValue((int) i, out var glyphIndex))
                    {
                        // 不在范围内的字符，居然可以找到对应的字形索引
                        _ = glyphIndex;
                        Debugger.Break();
                    }
                    else
                    {
                        // 不在范围内的字符，预期找不到对应的字形索引
                    }
                }
            }

            bool IsInUnicodeRange(uint codepoint)
            {
                foreach (var unicodeRange in unicodeRanges)
                {
                    if (codepoint >= unicodeRange.First && codepoint <= unicodeRange.Last)
                    {
                        return true;
                    }
                }

                return false;
            }
        }
    }
```

如果大家对 WPF 的 CharacterToGlyphMap 实现逻辑感兴趣，请参阅 [读 WPF 源代码 了解获取 GlyphTypeface 的 CharacterToGlyphMap 的数量耗时原因](https://blog.lindexi.com/post/%E8%AF%BB-WPF-%E6%BA%90%E4%BB%A3%E7%A0%81-%E4%BA%86%E8%A7%A3%E8%8E%B7%E5%8F%96-GlyphTypeface-%E7%9A%84-CharacterToGlyphMap-%E7%9A%84%E6%95%B0%E9%87%8F%E8%80%97%E6%97%B6%E5%8E%9F%E5%9B%A0.html )
<!-- [读 WPF 源代码 了解获取 GlyphTypeface 的 CharacterToGlyphMap 的数量耗时原因 - lindexi - 博客园](https://www.cnblogs.com/lindexi/p/19114691 ) -->

本文代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/e725768ffd131bda55560242748f4f26400ef006/DirectX/DWrite/JallwirekebalaChelchelkonuya) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/e725768ffd131bda55560242748f4f26400ef006/DirectX/DWrite/JallwirekebalaChelchelkonuya) 上，可以使用如下命令行拉取代码。我整个代码仓库比较庞大，使用以下命令行可以进行部分拉取，拉取速度比较快

先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin e725768ffd131bda55560242748f4f26400ef006
```

以上使用的是国内的 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码，将 gitee 源换成 github 源进行拉取代码。如果依然拉取不到代码，可以发邮件向我要代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin e725768ffd131bda55560242748f4f26400ef006
```

获取代码之后，进入 DirectX/DWrite/JallwirekebalaChelchelkonuya 文件夹，即可获取到源代码

更多技术博客，请参阅 [博客导航](https://blog.lindexi.com/post/%E5%8D%9A%E5%AE%A2%E5%AF%BC%E8%88%AA.html )

参考文档：

[IDWriteFontFace1::GetUnicodeRanges (dwrite_1.h) - Win32 apps - Microsoft Learn](https://learn.microsoft.com/en-us/windows/win32/api/dwrite_1/nf-dwrite_1-idwritefontface1-getunicoderanges )
