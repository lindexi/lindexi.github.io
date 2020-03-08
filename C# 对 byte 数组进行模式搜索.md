# C# 对 byte 数组进行模式搜索

本文告诉大家几个方法从 byte 数组找到对应的相同序列的数组

<!--more-->
<!-- CreateTime:2019/8/31 16:55:58 -->


最简单的方法是进行数值判断，但是代码最少是使用Linq ，效率比较高是使用 Boyer-Moore 算法，下面就告诉大家几个算法的代码

## 判断数值

```csharp
    class ByteArrayRocks
    {
        public static IEnumerable<int> IndexOf(byte[] source, int start, byte[] pattern)
        {
            if (IsEmptyLocate(source, start, pattern))
            {
                yield break;
            }

            for (int i = start; i < source.Length; i++)
            {
                if (!IsMatch(source, i, pattern))
                {
                    continue;
                }

                yield return i;
            }
        }

        private static readonly int[] Empty = new int[0];

        private static bool IsMatch(byte[] array, int position, byte[] candidate)
        {
            if (candidate.Length > (array.Length - position))
            {
                return false;
            }

            for (int i = 0; i < candidate.Length; i++)
            {
                if (array[position + i] != candidate[i])
                {
                    return false;
                }
            }

            return true;
        }

        private static bool IsEmptyLocate(byte[] array, int start, byte[] candidate)
        {
            return array == null
                   || candidate == null
                   || array.Length == 0
                   || array.Length < start
                   || candidate.Length == 0
                   || candidate.Length + start > array.Length;
        }
    }
```

这是最简单的方法，参见 https://stackoverflow.com/a/283648/6116637

## linq 

这个方法的代码最少

```csharp
    class LinqArraySearch
    {
        public static IEnumerable<int> IndexOf(byte[] source, int start, byte[] pattern)
        {
            for (int i = start; i < source.Length; i++)
            {
                if (source.Skip(i).Take(pattern.Length).SequenceEqual(pattern))
                {
                    yield return i;
                }
            }
        }
    }
```

## Boyer-Moore-Horspool 搜索

这是最快的方法

```csharp
    class BoyerMooreHorspool
    {
        public static IEnumerable<long> IndexesOf(byte[] source, int start, byte[] pattern)
        {
            if (source == null)
            {
                throw new ArgumentNullException(nameof(source));
            }

            if (pattern == null)
            {
                throw new ArgumentNullException(nameof(pattern));
            }

            long valueLength = source.LongLength;
            long patternLength = pattern.LongLength;

            if ((valueLength == 0) || (patternLength == 0) || (patternLength > valueLength))
            {
                yield break;
            }

            var badCharacters = new long[256];

            for (var i = 0; i < 256; i++)
            {
                badCharacters[i] = patternLength;
            }

            var lastPatternByte = patternLength - 1;

            for (long i = 0; i < lastPatternByte; i++)
            {
                badCharacters[pattern[i]] = lastPatternByte - i;
            }

            long index = start;

            while (index <= valueLength - patternLength)
            {
                for (var i = lastPatternByte; source[index + i] == pattern[i]; i--)
                {
                    if (i == 0)
                    {
                        yield return index;
                        break;
                    }
                }

                index += badCharacters[source[index + lastPatternByte]];
            }
        }
    }

```

参见：https://stackoverflow.com/q/16252518/6116637

测试了三个方法的性能，请看下面

``` ini

BenchmarkDotNet=v0.10.14, OS=Windows 10.0.17134
Intel Core i7-6700 CPU 3.40GHz (Skylake), 1 CPU, 8 logical and 4 physical cores
.NET Core SDK=2.1.202
  [Host]     : .NET Core 2.0.9 (CoreCLR 4.6.26614.01, CoreFX 4.6.26614.01), 64bit RyuJIT
  DefaultJob : .NET Core 2.0.9 (CoreCLR 4.6.26614.01, CoreFX 4.6.26614.01), 64bit RyuJIT


```

|                                      Method |        Mean |      Error |     StdDev |
|-------------------------------------------- |------------:|-----------:|-----------:|
|                                        Linq | 13,332.8 us | 251.782 us | 466.694 us |
|                              ByteArrayRocks |    371.3 us |   7.327 us |  14.462 us |
|                          BoyerMooreHorspool |    108.3 us |   1.153 us |   1.079 us |

其他方法请看下面

使用不安全代码的 Boyer Moore 算法 [C# High Performance Boyer Moore Byte Array Search Algorithm](https://gist.github.com/mjs3339/0772431281093f1bca1fce2f2eca527d )

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
