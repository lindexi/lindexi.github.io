---
title: 读 WPF 源代码 了解获取 GlyphTypeface 的 CharacterToGlyphMap 的数量耗时原因
description: 在我的一次应用性能分析中，我发现了尝试获取 GlyphTypeface 的 CharacterToGlyphMap 属性的数量时，存在很大的耗时。经过阅读 WPF 源代码，我了解到了其中的原因，本文将和大家从底层源代码分析原因和给出解决方法
tags: WPF
category: 
---

<!-- 发布 -->
<!-- 博客 -->

在 WPF 里面的 GlyphTypeface 表示字体的字形信息，通过 `GlyphTypeface.CharacterToGlyphMap` 属性可以将给定的字符映射到字形索引，这个属性是一个 `IDictionary<int, ushort>` 结构，其定义如下

```csharp
    public class GlyphTypeface : ITypefaceMetrics, ISupportInitialize
    {
        ...

        public IDictionary<int, ushort> CharacterToGlyphMap
        {
            get
            {
                CheckInitialized(); // This can only be called on fully initialized GlyphTypeface
                return _fontFace.CharacterMap;
            }
        }

        private FontFaceLayoutInfo          _fontFace;
        ...
    }
```

咋看起来没问题，实际调用的时候，通过 CharacterToGlyphMap 的 TryGetValue 方法获取传入的字符对应的字形索引也是十分快速，没有什么耗时。唯独只有调用 `CharacterToGlyphMap.Count` 属性时，才能测量到很大的耗时

这是为什么呢，让咱继续点开 `_fontFace.CharacterMap` 属性的定义，其代码如下

```csharp
    internal sealed class FontFaceLayoutInfo
    {
        ...

        internal IntMap CharacterMap
        {
            get
            {
                return _cmap;
            }
        }
        ...
    }
```

可见是返回了 `IntMap` 类型的 `_cmap` 字段，看到这里，似乎非常清真

继续看看 `IntMap` 类型的实现

```csharp
        internal sealed class IntMap : IDictionary<int, ushort>
        {
            ...
            public int Count
            {
                get { return CMap.Count; }
            }

            private Dictionary<int, ushort> CMap
            {
            	...
            }
            ...
        }
```

看起来其中核心就在于 `CMap` 的 get 函数里，继续看看此属性的实现

```csharp
        internal sealed class IntMap : IDictionary<int, ushort>
        {
            ...
            private Dictionary<int, ushort> CMap
            {
                get
                {
                    if (_cmap == null)
                    {
                        lock (this)
                        {
                            if (_cmap == null)
                            {
                                _cmap = new Dictionary<int, ushort>();
                                ushort glyphIndex;
                                for (int codePoint = 0; codePoint <= FontFamilyMap.LastUnicodeScalar; ++codePoint)
                                {
                                    if (TryGetValue(codePoint, out glyphIndex))
                                    {
                                        _cmap.Add(codePoint, glyphIndex);
                                    }
                                }
                            }
                        }
                    }
                    return _cmap;
                }
            }

            private Dictionary<int, ushort>     _cmap;

            ...
        }

    public class FontFamilyMap
    {
         ...

        internal const int LastUnicodeScalar = 0x10ffff;
         ...
     }
```

可以看到，此时如果调用 `CharacterToGlyphMap.Count` 属性，将会导致 `_cmap` 字段被初始化。而初始化的过程是采用一个巨大的循环，足足有 0x10ffff 的百万次循环调用 TryGetValue 方法创建的字典

即使 TryGetValue 方法速度再快，但是循环本身将会调用 1114111 （0x10ffff） 百万次，这就是耗时的原因

按照当前的代码，直接调用 `CharacterToGlyphMap.Count` 属性是非常亏的，将会导致 `_cmap` 字段，初始化此字典需要经过百万次的循环，自然性能很差。但很显然，这是一个很好做的优化点，只需要绕开字典初始化，直接获取数量即可

既然看起来这是一个很好的优化点，自然我就将其优化了： <https://github.com/dotnet/wpf/pull/11139>

对于业务端开发者，等不及 WPF 的更新的伙伴们，可以直接使用 `GlyphTypeface.GlyphCount` 属性代替 `CharacterToGlyphMap.Count` 属性即可

```csharp
    public class GlyphTypeface : ITypefaceMetrics, ISupportInitialize
    {
        ...

        public int GlyphCount
        {
            get
            {
                CheckInitialized(); // This can only be called on fully initialized GlyphTypeface
                int glyphCount;

                MS.Internal.Text.TextInterface.FontFace fontFaceDWrite = _font.GetFontFace();
                try
                {
                    glyphCount = fontFaceDWrite.GlyphCount;
                }
                finally
                {
                    fontFaceDWrite.Release();
                }

                return glyphCount;
            }
        }

        ...
    }
```

通过以上代码可见 GlyphCount 属性是直接获取的，基本测量不出耗时
