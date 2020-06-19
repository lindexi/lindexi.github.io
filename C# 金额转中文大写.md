# C# 金额转中文大写

今天看到一个库是把金额转中文大写，看起来很容易，所以我就自己写了。

<!--more-->
<!-- CreateTime:2018/4/29 9:50:38 -->


创建的项目是创建一个 dot net core 的项目，实际上这个项目可以创建为 Stand 的。

首先创建类，这个类的构造传入一个 double 作为人民币

```csharp
    public class Money
    {
        public Money(double money)
        {
            _money = money;
        }

        private double _money;
  }
```

然后创建方法转换

```csharp
        public string ToCapital()
        {
            if (Math.Abs(_money) < 0.0001)
            {
                return "零元";
            }

            var str = GetIntPart();
            GetDecimalPart(str);
            return str.ToString();
        }
```

其中 GetIntPart 是转换小数点前的部分，小数点之后的使用 GetDecimalPart 计算。

然后创建一些使用的数组

```csharp
        private static readonly List<char> Uppers = new List<char>()
        {
            '零',
            '壹',
            '贰',
            '叁',
            '肆',
            '伍',
            '陆',
            '柒',
            '捌',
            '玖'
        };

        private static readonly List<char> Units = new List<char>()
        {
            '分',
            '角'
        };

        private static readonly List<char> Grees = new List<char>()
        {
            '元',
            '拾',
            '佰',
            '仟',
            '万',
            '拾',
            '佰',
            '仟',
            '亿',
            '拾',
            '佰',
            '仟',
            '万',
            '拾',
            '佰'
        };
```

转换小数点前的代码

```csharp
           StringBuilder str = new StringBuilder();
            var money = _money;
        
            for (int i = 0; money > 0.99999; i++)
            {
                var n = (int) (money % 10);
                str.Insert(0,Grees[i]);
                str.Insert(0,Uppers[n]);
                money = money / 10;
                money = money - n / 10.0;
            }
```

但是这样转换得到的存在一些零，如输入 100 会输出 壹佰零拾零元 ，所以需要对输出转换

```csharp
        private StringBuilder GetIntPart()
        {
            StringBuilder str = new StringBuilder();
            var money = _money;

            for (int i = 0; money > 0.99999; i++)
            {
                var n = (int) (money % 10);
                str.Insert(0, Grees[i]);
                str.Insert(0, Uppers[n]);
                money = money / 10;
                money = money - n / 10.0;
            }

            str = str.Replace("零亿", "亿零");
            str = str.Replace("零万", "万零");

            str = str.Replace("零拾", "零");
            str = str.Replace("零佰", "零");
            str = str.Replace("零仟", "零");

            str = str.Replace("零零", "零");
            str = str.Replace("零零", "零");

            str = str.Replace("零亿", "亿");
            str = str.Replace("零万", "万");
            str = str.Replace("零元", "元");

            return str;
        }

```

转换小数的代码

```csharp
        private void GetDecimalPart(StringBuilder str)
        {
            var money = _money * 100;
            for (int i = 0; i < 2; i++)
            {
                var n = (int) (money % 10);
                if (n != 0)
                {
                    str.Insert(0, Units[i]);
                    str.Insert(0, Uppers[n]);
                }

                money = money / 10;
            }
        }

```

所有代码

<script src='https://gitee.com/lindexi/codes/w6bxlue9o14rv5nscjyhf20/widget_preview?title=Money'></script>

代码请看 https://gitee.com/lindexi/codes/w6bxlue9o14rv5nscjyhf20

另外有一个更全的库，请看 [zmjack/Chinese: 中文解析通用工具。包括拼音，简繁转换，数字读法，货币读法。](https://github.com/zmjack/Chinese )


```csharp
// "mian3 fei4，kua4 ping2 tai1，kai1 yuan2！"
Pinyin.GetString("免费，跨平台，开源！", PinyinFormat.Default);

// "mian fei，kua ping tai，kai yuan！"
Pinyin.GetString("免费，跨平台，开源！", PinyinFormat.WithoutTone);

// "miǎn fèi，kuà píng tāi，kāi yuán！"
Pinyin.GetString("免费，跨平台，开源！", PinyinFormat.PhoneticSymbol);

var options = new ChineseNumberOptions { Simplified = false, Upper = false };
ChineseNumber.GetString(10_0001, options);    // "一十万零一"
ChineseNumber.GetString(10_0101, options);    // "一十万零一百零一"
ChineseNumber.GetString(10_1001, options);    // "一十万一千零一"
ChineseNumber.GetString(10_1010, options);    // "一十万一千零一十"

ChineseCurrency.GetString(10_0001, options);       // "一十万零一元整"
ChineseCurrency.GetString(10_0101, options);       // "一十万零一百零一元整"
ChineseCurrency.GetString(10_1001, options);       // "一十万一千零一元整"
ChineseCurrency.GetString(10_1010, options);       // "一十万一千零一十元整"
ChineseCurrency.GetString(10_0001.2m, options);    // "一十万零一元二角整"
ChineseCurrency.GetString(10_0001.23m, options);   // "一十万零一元二角三分"
ChineseCurrency.GetString(10_0001.03m, options);   // "一十万零一元零三分"
```

参见：[src/Money.php · 趋势软件/capital - 码云 Gitee.com](https://gitee.com/trendsoftorg/capital/blob/master/src/Money.php )

![](https://i.loli.net/2018/04/08/5ac9ffa67477f.jpg)

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
