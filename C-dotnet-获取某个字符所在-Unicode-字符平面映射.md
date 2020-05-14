
# C# dotnet 获取某个字符所在 Unicode 字符平面映射

在 dotnet 里面可以通过安装 System.Text.Encodings.Web 库拿到 UnicodeRanges 这个包含了 Unicode 标准的平面映射。但是我还没有找到如何判断一个字符是落在哪个平面的逻辑，本文就告诉大家一个可以使用的方法，这个方法同时稍微兼顾了性能

<!--more-->


<!-- CreateTime:5/14/2020 3:08:23 PM -->

<!-- 发布 -->

在 System.Text.Encodings.Web 的 UnicodeRanges 其实是根据标准生成的代码，源代码请看 [https://github.com/dotnet/runtime/blob/5372ee9dbe48058ca8d3591763e989d3b2e65581/src/libraries/System.Text.Encodings.Web/src/System/Text/Unicode/UnicodeRanges.generated.cs](https://github.com/dotnet/runtime/blob/5372ee9dbe48058ca8d3591763e989d3b2e65581/src/libraries/System.Text.Encodings.Web/src/System/Text/Unicode/UnicodeRanges.generated.cs)

本文根据 UnicodeRanges 的定义写出了一个列表如下

```csharp
        /// <summary>
        /// 这个列表已经是按照 FirstCodePoint 排过顺序的
        /// </summary>
        private static List<UnicodeRangeInfo> UnicodeRangeInfoList { get; } = new List<UnicodeRangeInfo>()
        {
            //new UnicodeRangeInfo("None",UnicodeRanges.None),
            //new UnicodeRangeInfo("All",UnicodeRanges.All),
            new UnicodeRangeInfo("BasicLatin", UnicodeRanges.BasicLatin),
            new UnicodeRangeInfo("Latin1Supplement", UnicodeRanges.Latin1Supplement),
            new UnicodeRangeInfo("LatinExtendedA", UnicodeRanges.LatinExtendedA),
            new UnicodeRangeInfo("LatinExtendedB", UnicodeRanges.LatinExtendedB),
            new UnicodeRangeInfo("IpaExtensions", UnicodeRanges.IpaExtensions),
            new UnicodeRangeInfo("SpacingModifierLetters", UnicodeRanges.SpacingModifierLetters),
            new UnicodeRangeInfo("CombiningDiacriticalMarks", UnicodeRanges.CombiningDiacriticalMarks),
            new UnicodeRangeInfo("GreekandCoptic", UnicodeRanges.GreekandCoptic),
            new UnicodeRangeInfo("Cyrillic", UnicodeRanges.Cyrillic),
            new UnicodeRangeInfo("CyrillicSupplement", UnicodeRanges.CyrillicSupplement),
            new UnicodeRangeInfo("Armenian", UnicodeRanges.Armenian),
            new UnicodeRangeInfo("Hebrew", UnicodeRanges.Hebrew),
            new UnicodeRangeInfo("Arabic", UnicodeRanges.Arabic),
            new UnicodeRangeInfo("Syriac", UnicodeRanges.Syriac),
            new UnicodeRangeInfo("ArabicSupplement", UnicodeRanges.ArabicSupplement),
            new UnicodeRangeInfo("Thaana", UnicodeRanges.Thaana),
            new UnicodeRangeInfo("NKo", UnicodeRanges.NKo),
            new UnicodeRangeInfo("Samaritan", UnicodeRanges.Samaritan),
            new UnicodeRangeInfo("Mandaic", UnicodeRanges.Mandaic),
            new UnicodeRangeInfo("SyriacSupplement", UnicodeRanges.SyriacSupplement),
            new UnicodeRangeInfo("ArabicExtendedA", UnicodeRanges.ArabicExtendedA),
            new UnicodeRangeInfo("Devanagari", UnicodeRanges.Devanagari),
            new UnicodeRangeInfo("Bengali", UnicodeRanges.Bengali),
            new UnicodeRangeInfo("Gurmukhi", UnicodeRanges.Gurmukhi),
            new UnicodeRangeInfo("Gujarati", UnicodeRanges.Gujarati),
            new UnicodeRangeInfo("Oriya", UnicodeRanges.Oriya),
            new UnicodeRangeInfo("Tamil", UnicodeRanges.Tamil),
            new UnicodeRangeInfo("Telugu", UnicodeRanges.Telugu),
            new UnicodeRangeInfo("Kannada", UnicodeRanges.Kannada),
            new UnicodeRangeInfo("Malayalam", UnicodeRanges.Malayalam),
            new UnicodeRangeInfo("Sinhala", UnicodeRanges.Sinhala),
            new UnicodeRangeInfo("Thai", UnicodeRanges.Thai),
            new UnicodeRangeInfo("Lao", UnicodeRanges.Lao),
            new UnicodeRangeInfo("Tibetan", UnicodeRanges.Tibetan),
            new UnicodeRangeInfo("Myanmar", UnicodeRanges.Myanmar),
            new UnicodeRangeInfo("Georgian", UnicodeRanges.Georgian),
            new UnicodeRangeInfo("HangulJamo", UnicodeRanges.HangulJamo),
            new UnicodeRangeInfo("Ethiopic", UnicodeRanges.Ethiopic),
            new UnicodeRangeInfo("EthiopicSupplement", UnicodeRanges.EthiopicSupplement),
            new UnicodeRangeInfo("Cherokee", UnicodeRanges.Cherokee),
            new UnicodeRangeInfo("UnifiedCanadianAboriginalSyllabics",
                UnicodeRanges.UnifiedCanadianAboriginalSyllabics),
            new UnicodeRangeInfo("Ogham", UnicodeRanges.Ogham),
            new UnicodeRangeInfo("Runic", UnicodeRanges.Runic),
            new UnicodeRangeInfo("Tagalog", UnicodeRanges.Tagalog),
            new UnicodeRangeInfo("Hanunoo", UnicodeRanges.Hanunoo),
            new UnicodeRangeInfo("Buhid", UnicodeRanges.Buhid),
            new UnicodeRangeInfo("Tagbanwa", UnicodeRanges.Tagbanwa),
            new UnicodeRangeInfo("Khmer", UnicodeRanges.Khmer),
            new UnicodeRangeInfo("Mongolian", UnicodeRanges.Mongolian),
            new UnicodeRangeInfo("UnifiedCanadianAboriginalSyllabicsExtended",
                UnicodeRanges.UnifiedCanadianAboriginalSyllabicsExtended),
            new UnicodeRangeInfo("Limbu", UnicodeRanges.Limbu),
            new UnicodeRangeInfo("TaiLe", UnicodeRanges.TaiLe),
            new UnicodeRangeInfo("NewTaiLue", UnicodeRanges.NewTaiLue),
            new UnicodeRangeInfo("KhmerSymbols", UnicodeRanges.KhmerSymbols),
            new UnicodeRangeInfo("Buginese", UnicodeRanges.Buginese),
            new UnicodeRangeInfo("TaiTham", UnicodeRanges.TaiTham),
            new UnicodeRangeInfo("CombiningDiacriticalMarksExtended", UnicodeRanges.CombiningDiacriticalMarksExtended),
            new UnicodeRangeInfo("Balinese", UnicodeRanges.Balinese),
            new UnicodeRangeInfo("Sundanese", UnicodeRanges.Sundanese),
            new UnicodeRangeInfo("Batak", UnicodeRanges.Batak),
            new UnicodeRangeInfo("Lepcha", UnicodeRanges.Lepcha),
            new UnicodeRangeInfo("OlChiki", UnicodeRanges.OlChiki),
            new UnicodeRangeInfo("CyrillicExtendedC", UnicodeRanges.CyrillicExtendedC),
            new UnicodeRangeInfo("GeorgianExtended", UnicodeRanges.GeorgianExtended),
            new UnicodeRangeInfo("SundaneseSupplement", UnicodeRanges.SundaneseSupplement),
            new UnicodeRangeInfo("VedicExtensions", UnicodeRanges.VedicExtensions),
            new UnicodeRangeInfo("PhoneticExtensions", UnicodeRanges.PhoneticExtensions),
            new UnicodeRangeInfo("PhoneticExtensionsSupplement", UnicodeRanges.PhoneticExtensionsSupplement),
            new UnicodeRangeInfo("CombiningDiacriticalMarksSupplement",
                UnicodeRanges.CombiningDiacriticalMarksSupplement),
            new UnicodeRangeInfo("LatinExtendedAdditional", UnicodeRanges.LatinExtendedAdditional),
            new UnicodeRangeInfo("GreekExtended", UnicodeRanges.GreekExtended),
            new UnicodeRangeInfo("GeneralPunctuation", UnicodeRanges.GeneralPunctuation),
            new UnicodeRangeInfo("SuperscriptsandSubscripts", UnicodeRanges.SuperscriptsandSubscripts),
            new UnicodeRangeInfo("CurrencySymbols", UnicodeRanges.CurrencySymbols),
            new UnicodeRangeInfo("CombiningDiacriticalMarksforSymbols",
                UnicodeRanges.CombiningDiacriticalMarksforSymbols),
            new UnicodeRangeInfo("LetterlikeSymbols", UnicodeRanges.LetterlikeSymbols),
            new UnicodeRangeInfo("NumberForms", UnicodeRanges.NumberForms),
            new UnicodeRangeInfo("Arrows", UnicodeRanges.Arrows),
            new UnicodeRangeInfo("MathematicalOperators", UnicodeRanges.MathematicalOperators),
            new UnicodeRangeInfo("MiscellaneousTechnical", UnicodeRanges.MiscellaneousTechnical),
            new UnicodeRangeInfo("ControlPictures", UnicodeRanges.ControlPictures),
            new UnicodeRangeInfo("OpticalCharacterRecognition", UnicodeRanges.OpticalCharacterRecognition),
            new UnicodeRangeInfo("EnclosedAlphanumerics", UnicodeRanges.EnclosedAlphanumerics),
            new UnicodeRangeInfo("BoxDrawing", UnicodeRanges.BoxDrawing),
            new UnicodeRangeInfo("BlockElements", UnicodeRanges.BlockElements),
            new UnicodeRangeInfo("GeometricShapes", UnicodeRanges.GeometricShapes),
            new UnicodeRangeInfo("MiscellaneousSymbols", UnicodeRanges.MiscellaneousSymbols),
            new UnicodeRangeInfo("Dingbats", UnicodeRanges.Dingbats),
            new UnicodeRangeInfo("MiscellaneousMathematicalSymbolsA", UnicodeRanges.MiscellaneousMathematicalSymbolsA),
            new UnicodeRangeInfo("SupplementalArrowsA", UnicodeRanges.SupplementalArrowsA),
            new UnicodeRangeInfo("BraillePatterns", UnicodeRanges.BraillePatterns),
            new UnicodeRangeInfo("SupplementalArrowsB", UnicodeRanges.SupplementalArrowsB),
            new UnicodeRangeInfo("MiscellaneousMathematicalSymbolsB", UnicodeRanges.MiscellaneousMathematicalSymbolsB),
            new UnicodeRangeInfo("SupplementalMathematicalOperators", UnicodeRanges.SupplementalMathematicalOperators),
            new UnicodeRangeInfo("MiscellaneousSymbolsandArrows", UnicodeRanges.MiscellaneousSymbolsandArrows),
            new UnicodeRangeInfo("Glagolitic", UnicodeRanges.Glagolitic),
            new UnicodeRangeInfo("LatinExtendedC", UnicodeRanges.LatinExtendedC),
            new UnicodeRangeInfo("Coptic", UnicodeRanges.Coptic),
            new UnicodeRangeInfo("GeorgianSupplement", UnicodeRanges.GeorgianSupplement),
            new UnicodeRangeInfo("Tifinagh", UnicodeRanges.Tifinagh),
            new UnicodeRangeInfo("EthiopicExtended", UnicodeRanges.EthiopicExtended),
            new UnicodeRangeInfo("CyrillicExtendedA", UnicodeRanges.CyrillicExtendedA),
            new UnicodeRangeInfo("SupplementalPunctuation", UnicodeRanges.SupplementalPunctuation),
            new UnicodeRangeInfo("CjkRadicalsSupplement", UnicodeRanges.CjkRadicalsSupplement),
            new UnicodeRangeInfo("KangxiRadicals", UnicodeRanges.KangxiRadicals),
            new UnicodeRangeInfo("IdeographicDescriptionCharacters", UnicodeRanges.IdeographicDescriptionCharacters),
            new UnicodeRangeInfo("CjkSymbolsandPunctuation", UnicodeRanges.CjkSymbolsandPunctuation),
            new UnicodeRangeInfo("Hiragana", UnicodeRanges.Hiragana),
            new UnicodeRangeInfo("Katakana", UnicodeRanges.Katakana),
            new UnicodeRangeInfo("Bopomofo", UnicodeRanges.Bopomofo),
            new UnicodeRangeInfo("HangulCompatibilityJamo", UnicodeRanges.HangulCompatibilityJamo),
            new UnicodeRangeInfo("Kanbun", UnicodeRanges.Kanbun),
            new UnicodeRangeInfo("BopomofoExtended", UnicodeRanges.BopomofoExtended),
            new UnicodeRangeInfo("CjkStrokes", UnicodeRanges.CjkStrokes),
            new UnicodeRangeInfo("KatakanaPhoneticExtensions", UnicodeRanges.KatakanaPhoneticExtensions),
            new UnicodeRangeInfo("EnclosedCjkLettersandMonths", UnicodeRanges.EnclosedCjkLettersandMonths),
            new UnicodeRangeInfo("CjkCompatibility", UnicodeRanges.CjkCompatibility),
            new UnicodeRangeInfo("CjkUnifiedIdeographsExtensionA", UnicodeRanges.CjkUnifiedIdeographsExtensionA),
            new UnicodeRangeInfo("YijingHexagramSymbols", UnicodeRanges.YijingHexagramSymbols),
            new UnicodeRangeInfo("CjkUnifiedIdeographs", UnicodeRanges.CjkUnifiedIdeographs),
            new UnicodeRangeInfo("YiSyllables", UnicodeRanges.YiSyllables),
            new UnicodeRangeInfo("YiRadicals", UnicodeRanges.YiRadicals),
            new UnicodeRangeInfo("Lisu", UnicodeRanges.Lisu),
            new UnicodeRangeInfo("Vai", UnicodeRanges.Vai),
            new UnicodeRangeInfo("CyrillicExtendedB", UnicodeRanges.CyrillicExtendedB),
            new UnicodeRangeInfo("Bamum", UnicodeRanges.Bamum),
            new UnicodeRangeInfo("ModifierToneLetters", UnicodeRanges.ModifierToneLetters),
            new UnicodeRangeInfo("LatinExtendedD", UnicodeRanges.LatinExtendedD),
            new UnicodeRangeInfo("SylotiNagri", UnicodeRanges.SylotiNagri),
            new UnicodeRangeInfo("CommonIndicNumberForms", UnicodeRanges.CommonIndicNumberForms),
            new UnicodeRangeInfo("Phagspa", UnicodeRanges.Phagspa),
            new UnicodeRangeInfo("Saurashtra", UnicodeRanges.Saurashtra),
            new UnicodeRangeInfo("DevanagariExtended", UnicodeRanges.DevanagariExtended),
            new UnicodeRangeInfo("KayahLi", UnicodeRanges.KayahLi),
            new UnicodeRangeInfo("Rejang", UnicodeRanges.Rejang),
            new UnicodeRangeInfo("HangulJamoExtendedA", UnicodeRanges.HangulJamoExtendedA),
            new UnicodeRangeInfo("Javanese", UnicodeRanges.Javanese),
            new UnicodeRangeInfo("MyanmarExtendedB", UnicodeRanges.MyanmarExtendedB),
            new UnicodeRangeInfo("Cham", UnicodeRanges.Cham),
            new UnicodeRangeInfo("MyanmarExtendedA", UnicodeRanges.MyanmarExtendedA),
            new UnicodeRangeInfo("TaiViet", UnicodeRanges.TaiViet),
            new UnicodeRangeInfo("MeeteiMayekExtensions", UnicodeRanges.MeeteiMayekExtensions),
            new UnicodeRangeInfo("EthiopicExtendedA", UnicodeRanges.EthiopicExtendedA),
            new UnicodeRangeInfo("LatinExtendedE", UnicodeRanges.LatinExtendedE),
            new UnicodeRangeInfo("CherokeeSupplement", UnicodeRanges.CherokeeSupplement),
            new UnicodeRangeInfo("MeeteiMayek", UnicodeRanges.MeeteiMayek),
            new UnicodeRangeInfo("HangulSyllables", UnicodeRanges.HangulSyllables),
            new UnicodeRangeInfo("HangulJamoExtendedB", UnicodeRanges.HangulJamoExtendedB),
            new UnicodeRangeInfo("CjkCompatibilityIdeographs", UnicodeRanges.CjkCompatibilityIdeographs),
            new UnicodeRangeInfo("AlphabeticPresentationForms", UnicodeRanges.AlphabeticPresentationForms),
            new UnicodeRangeInfo("ArabicPresentationFormsA", UnicodeRanges.ArabicPresentationFormsA),
            new UnicodeRangeInfo("VariationSelectors", UnicodeRanges.VariationSelectors),
            new UnicodeRangeInfo("VerticalForms", UnicodeRanges.VerticalForms),
            new UnicodeRangeInfo("CombiningHalfMarks", UnicodeRanges.CombiningHalfMarks),
            new UnicodeRangeInfo("CjkCompatibilityForms", UnicodeRanges.CjkCompatibilityForms),
            new UnicodeRangeInfo("SmallFormVariants", UnicodeRanges.SmallFormVariants),
            new UnicodeRangeInfo("ArabicPresentationFormsB", UnicodeRanges.ArabicPresentationFormsB),
            new UnicodeRangeInfo("HalfwidthandFullwidthForms", UnicodeRanges.HalfwidthandFullwidthForms),
            new UnicodeRangeInfo("Specials", UnicodeRanges.Specials),
        };
```

如果不信这个列表已经排好序，请使用下面方法测试一下

```csharp
        /// <summary>
        /// 按照 FirstCodePoint 排序，看顺序是否是对的，这个方法仅是让你知道这个列表是排序的
        /// </summary>
        /// <returns></returns>
        internal static bool CheckSort()
        {
            var t = UnicodeRangeInfoList.ToList();
            t.Sort((a, b) => a.UnicodeRange.FirstCodePoint.CompareTo(b.UnicodeRange.FirstCodePoint));

            for (var i = 0; i < UnicodeRangeInfoList.Count; i++)
            {
                if (!ReferenceEquals(t[i], UnicodeRangeInfoList[i]))
                {
                    return false;
                }

            }

            return true;
        }
```

如果上面方法返回 true 那么证明排序之后的列表和当前的列表是完全相同

这里的 UnicodeRangeInfo 是自己定义的类，可以让平面的名字和值关联

```csharp
        private class UnicodeRangeInfo
        {
            public UnicodeRangeInfo(string unicodeRangeName, UnicodeRange unicodeRange)
            {
                UnicodeRangeName = unicodeRangeName;
                UnicodeRange = unicodeRange;
            }

            public string UnicodeRangeName { get; }

            public UnicodeRange UnicodeRange { get; }
        }
```

在拿到一个字符的时候，根据 UnicodeRange 可以判断是否落在这个平面里面，从统计里面使用的字符大部分都是中文或英文字符，所以本文就根据这两个优化

```csharp
        private static UnicodeRangeInfo GetUnicodeRangeInfo(char ch)
        {
            // 这是英文 数字
            if (UnicodeRangeInfoList[0].UnicodeRange.Contain(ch))
            {
                // UnicodeRangeInfoList[0].UnicodeRange == UnicodeRanges.BasicLatin
                return UnicodeRangeInfoList[0];
            }

            // 119
            var currentIndex = 119;//准备二分，这个值刚好是 中日韩统一表意文字 CJK Unified Ideographs 
            var left = 0;
            var right = UnicodeRangeInfoList.Count;
            while (left <= right)
            {
                var unicodeRangeInfo = UnicodeRangeInfoList[currentIndex];
                var unicodeRange = unicodeRangeInfo.UnicodeRange;
                if (unicodeRange.Contain(ch))
                {
                    return unicodeRangeInfo;
                }
                else if (unicodeRange.FirstCodePoint < ch)
                {
                    // 不落在 unicodeRange 里面，同时比 FirstCodePoint 大的，一定比 unicodeRange.FirstCodePoint + unicodeRange.Length 大
                    left = currentIndex + 1;
                }
                else if (unicodeRange.FirstCodePoint > ch)
                {
                    right = currentIndex - 1;
                }

                currentIndex = (right + left) / 2;
            }

            return null;
        }
```

调用这个类可以使用三个方法，一个是获取字符所在的 UnicodeRange 可以用来判断多个字符是否落在一个平面，另一个是获取平面名。最后一个是根据所在平面获取平面名

```csharp
        /// <summary>
        /// 获取此字符所在范围
        /// </summary>
        /// <param name="ch"></param>
        /// <returns></returns>
        public static UnicodeRange GetUnicodeRange(char ch)
            => GetUnicodeRangeInfo(ch)?.UnicodeRange;

        /// <summary>
        /// 获取此字符所在范围名
        /// </summary>
        /// <param name="ch"></param>
        /// <returns></returns>
        public static string GetUnicodeRangeName(char ch)
            => GetUnicodeRangeInfo(ch)?.UnicodeRangeName;

        /// <summary>
        /// 获取字符范围名
        /// </summary>
        /// <param name="unicodeRange"></param>
        /// <returns></returns>
        public static string GetUnicodeRangeName(UnicodeRange unicodeRange) => UnicodeRangeInfoList
            .FirstOrDefault(temp => ReferenceEquals(temp.UnicodeRange, unicodeRange))?.UnicodeRangeName;
```

本文代码放在 [https://gist.github.com/lindexi/8945441c782488613fbc695d81a1bb85](https://gist.github.com/lindexi/8945441c782488613fbc695d81a1bb85) 欢迎小伙伴到 github 复制代码

<script src="https://gist.github.com/lindexi/8945441c782488613fbc695d81a1bb85.js"></script>

本文所有代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/40ebeab5576687c2d61e50df8a0733956894811a/CallnernawbawceKairwemwhejeene) 欢迎小伙伴访问

框架是否有提供差不多的方法？实际上有的，请看 [CharUnicodeInfo.GetUnicodeCategory 方法](https://docs.microsoft.com/zh-cn/dotnet/api/system.globalization.charunicodeinfo.getunicodecategory?view=netcore-3.1 )





<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。