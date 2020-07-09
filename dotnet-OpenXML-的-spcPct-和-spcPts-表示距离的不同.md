
# dotnet OpenXML 的 spcPct 和 spcPts 表示距离的不同

在 OpenXML 里面的文本排版里面使用到 spcPct （Spacing Percent） 和 spcPts（Spacing Points）两个不同的单位用来表示段前空白和段后空白以及行间距

<!--more-->


<!-- 发布 -->

在 Office 的文本排版里面，会在 spcAft (Space After) 段后空白空间以及 spcBef (Space Before) 段前空白空间等使用 spcPct （Spacing Percent）百分比空间 和 spcPts（Spacing Points） 固定点数空间表示空白空间

从命名上可以看出 spcPct （Spacing Percent）百分比空间 和 spcPts（Spacing Points） 固定点数空间的不同

## Spacing Percent

百分比空间相对复杂，这个单位会根据不同的业务采用不同的值，这个值本身只代表百分比，值的本身的数据可以加上百分比单位，如果加上单位了，那么则表示多少百分比。如果不加上单位，那么默认数据单位是千百分比，意思是用一千表示一个百分比 

如下面代码，添加了具体百分号单位，此时表示的就是百分比

```csharp
    <a:pPr …>
      <a:spcBef>
        <a:spcPct val="200%"/> 
      </a:spcBef> 
    </a:pPr>  
```

如下面代码，没有添加任何单位，此时表示的就是千倍的百分比


```csharp
    <a:pPr …>
      <a:spcBef>
        <a:spcPct val="200000"/> 
      </a:spcBef> 
    </a:pPr>  
```

上面两段代码的数值是等价的

在拿到百分比的值还需要知道这是相对于谁的大小才能计算出来

默认这个是相对于当前应用这个 a:rPr (RunProperties) 的 a:t (TextRun) 里面的字号最大的字号的值作为相对的百分比。也就是说在同一行的段落里面有不同的大小的字号，如有 10 号的和 20 号的，那么将使用最大的字号 20 号作为相对的值计算

关于百分比单位等请看 [Office Open XML 的测量单位](https://blog.lindexi.com/post/Office-Open-XML-%E7%9A%84%E6%B5%8B%E9%87%8F%E5%8D%95%E4%BD%8D.html )


## Spacing Points

固定点数空间的数值单位是百点也就是数值 100 表示 1 point 大小

这里的 point 和文本的字号的 point 是相等的含义

如下代码，由 `val="1500"` 表示段前空白长度是 15 point 大小

```csharp
    <a:pPr …>
        <a:spcBef>
                <a:spcPts val="1500"/>
        </a:spcBef>
    </a:pPr>
```

## 更多代码

附表示百分比的数值

```csharp
    /// <summary>
    /// 表示一个百分比数值
    /// </summary>
    public class Percentage
    {
        private const double Precision = 100000.0;

        /// <summary>
        /// 将一个openxml表示的百分比int值转换
        /// 每1000个单位代表1%
        /// <param name="value"></param>
        /// </summary>
        public Percentage(int value)
        {
            IntValue = value;
        }

        /// <summary>
        /// 将从一个double数值构建openxml表示的百分比
        /// 每0.01个double数值代表1%
        /// 会丢失精度
        /// <param name="value"></param>
        /// </summary>
        public static Percentage FromDouble(double value)
        {
            int v = (int) (value * Precision);
            return new Percentage(v);
        }

        /// <summary>
        /// openxml表示的百分比int值
        /// </summary>
        public int IntValue { get; }

        /// <summary>
        /// openxml表示的百分比double值
        /// 0-1
        /// </summary>
        public double DoubleValue => IntValue / Precision;

        /// <summary>
        /// 获取在指定范围内的double值
        /// </summary>
        /// <param name="min"></param>
        /// <param name="max"></param>
        /// <returns></returns>
        public double DoubleValueWithRange(double min, double max)
        {
            if (min > max)
            {
                throw new InvalidOperationException($"{nameof(max)}:{max} must greater than {nameof(min)}:{min}");
            }

            var value = IntValue / Precision;
            value = value > max ? max : value;
            value = value < min ? min : value;
            return value;
        }

        public override bool Equals(object obj)
        {
            if ((obj == null) || this.GetType() != obj.GetType())
            {
                return false;
            }
            else
            {
                Percentage p = (Percentage) obj;
                return IntValue == p.IntValue;
            }
        }

        public override int GetHashCode()
        {
            return this.IntValue.GetHashCode();
        }

        public static Percentage operator +(Percentage a) => a;
        public static Percentage operator -(Percentage a) => new Percentage(-a.IntValue);

        public static Percentage operator +(Percentage a, Percentage b)
            => new Percentage(a.IntValue + b.IntValue);

        public static Percentage operator -(Percentage a, Percentage b)
            => a + (-b);

        public static Percentage operator *(Percentage a, Percentage b)
            => new Percentage(a.IntValue * b.IntValue);

        public static Percentage operator /(Percentage a, Percentage b)
        {
            if (b.IntValue == 0)
            {
                throw new DivideByZeroException();
            }

            return new Percentage(a.IntValue / b.IntValue);
        }

        public static bool operator >(Percentage a, Percentage b)
        {
            return a.IntValue > b.IntValue;
        }

        public static bool operator <(Percentage a, Percentage b)
        {
            return b > a;
        }
    }
```

更多请看 

[RunProperties Class (DocumentFormat.OpenXml.Drawing)](https://docs.microsoft.com/en-us/dotnet/api/documentformat.openxml.drawing.runproperties?view=openxml-2.8.1 )






<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。