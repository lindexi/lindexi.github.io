# C# 搜索算法

本文主要讲C#搜索算法。

<!--more-->

<!--cdsn-->

## Bdf 算法

这算法是一个模糊的算法，用在用户在找一个他不确定的文本。

判断文本和匹配的字符是否有相同顺序，如果有，那么就是匹配。

假如我们有数据“abc”，匹配“abc”，那么，两个完全相对的字符串是匹配。

数据“aaacbc”，匹配“abc”，也是匹配，因为数据按照“abc”的顺序，算法不管数据有多长，只要数据存在和匹配相同的顺序，那么就匹配。

		
```
        /// <summary>
        /// 
        /// </summary>
        /// <param name="text">数据</param>
        /// <param name="str">匹配</param>
        public static bool Bdt(string text, string str)
        {
            int i = 0;
            bool reu = false;
            foreach (var temp in str)
            {
                reu = false;
                for (; i < text.Length; i++)
                {
                    if (temp == text[i])
                    {
                        reu = true;
                        break;
                    }
                }
            }
            return reu;
        }

```
