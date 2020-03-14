
# C# dontet Office Open XML Unit Converter

Here is my code to conver between OpenXML units.

<!--more-->


<!-- CreateTime:2020/3/12 18:11:17 -->

<!-- 发布 -->

## Define

If you use csharp language version lower than 7.0, please remove the readonly keyword from the C# code below

```csharp
    public readonly struct Cm
    {
        public Cm(double value)
        {
            Value = value;
        }

        public double Value { get; }
    }

    public readonly struct Dxa
    {
        public Dxa(double value)
        {
            Value = value;
        }

        public double Value { get; }
    }

    public readonly struct Emu
    {
        public Emu(double value)
        {
            Value = value;
        }

        public double Value { get; }
    }

    public readonly struct FiftiethsOfAPercent
    {
        public FiftiethsOfAPercent(double value)
        {
            Value = value;
        }

        public double Value { get; }
    }

    public readonly struct HalfPoint
    {
        public HalfPoint(double value)
        {
            Value = value;
        }

        public double Value { get; }
    }

    public readonly struct Inch
    {
        public Inch(double value)
        {
            Value = value;
        }

        public double Value { get; }
    }

    public readonly struct Mm
    {
        public Mm(double value)
        {
            Value = value;
        }

        public double Value { get; }
    }

    public readonly struct Pixel
    {
        public Pixel(double value)
        {
            Value = value;
        }

        public double Value { get; }
    }

    public readonly struct Pt
    {
        public Pt(double value)
        {
            Value = value;
        }

        /// <summary>
        /// 像素点的值
        /// </summary>
        public double Value { get; }
    }
```

## Convert

This is the code for converting different units of OpenXML

```csharp
    public static class UnitConverter
    {
        public const double DefaultDpi = 96;

        #region Dxa

        public static Pt ToPt(this Dxa dxa)
        {
            return new Pt(dxa.Value / 20);
        }

        public static Dxa ToDxa(this Pt pt)
        {
            return new Dxa(pt.Value * 20);
        }

        public static Inch ToInch(this Dxa dxa)
        {
            return new Inch(dxa.Value / 72);
        }

        public static Dxa ToDxa(this Inch inch)
        {
            return new Dxa(inch.Value * 72);
        }

        #endregion

        #region Mm

        public static Cm ToCm(this Mm mm)
        {
            return new Cm(mm.Value / 10);
        }

        public static Mm ToMm(this Cm cm)
        {
            return new Mm(cm.Value * 10);
        }

        #endregion

        #region Pt

        public static Cm ToCm(this Pt pt)
        {
            return pt.ToEmu().ToCm();
        }

        public static Pt ToPt(this Cm cm)
        {
            return cm.ToEmu().ToPt();
        }

        public static Mm ToMm(this Pt pt)
        {
            return pt.ToCm().ToMm();
        }

        public static Pt ToPt(this Mm mm)
        {
            return mm.ToCm().ToPt();
        }

        public static Pt ToPt(this HalfPoint halfPoint)
        {
            return new Pt(halfPoint.Value / 2);
        }

        public static HalfPoint ToHalfPoint(this Pt pt)
        {
            return new HalfPoint(pt.Value * 2);
        }


        public static Pixel ToPixel(this Pt pt)
        {
            return new Pixel(pt.Value / 72 * DefaultDpi);
        }

        public static Pt ToPoint(this Pixel px)
        {
            return new Pt(px.Value * 72 / DefaultDpi);
        }

        #endregion

        #region Emu

        public static Emu ToEmu(this Inch inch)
        {
            return new Emu(inch.Value * 914400);
        }

        public static Inch ToInch(this Emu emu)
        {
            return new Inch(emu.Value / 914400);
        }

        public static Emu ToEmu(this Cm cm)
        {
            return new Emu(cm.Value * 360000);
        }

        public static Cm ToCm(this Emu emu)
        {
            return new Cm(emu.Value / 360000);
        }

        public static Emu ToEmu(this Mm cm)
        {
            return new Emu(cm.Value * 36000);
        }

        public static Dxa ToDxa(this Emu emu)
        {
            return new Dxa(emu.Value / 635);
        }

        public static Emu ToEmu(this Dxa dxa)
        {
            return new Emu(dxa.Value * 635);
        }

        public static Mm ToMm(this Emu emu)
        {
            return new Mm(emu.Value / 36000);
        }


        public static Emu ToEmu(this Pixel px)
        {
            return new Emu(px.Value * 914400 / DefaultDpi);
        }

        public static Pixel ToPixel(this Emu emu)
        {
            return new Pixel(emu.Value / 914400 * DefaultDpi);
        }

        public static Emu ToEmu(this Pt pt)
        {
            return new Emu(pt.Value * 12700);
        }

        public static Pt ToPt(this Emu emu)
        {
            return new Pt(emu.Value / 12700);
        }

        #endregion
    }
```

## Test

The code in this article is not technical, but if you copy the code, it will reduce your time.

```csharp
        private void AssertDoubleEqual(double a, double b)
        {
            Assert.AreEqual(true, Math.Abs(a - b) < 0.001);
        }

                var pt = new Pt(1);

                AssertDoubleEqual(20, pt.ToDxa().Value);
                AssertDoubleEqual(12700, pt.ToEmu().Value);
                AssertDoubleEqual(0.3527777777777777, pt.ToMm().Value);
                AssertDoubleEqual(0.035277777777777776, pt.ToCm().Value);
                AssertDoubleEqual(0.01388888888, pt.ToEmu().ToInch().Value);

                var mm = new Mm(1000);

                AssertDoubleEqual(100, mm.ToCm().Value);
                AssertDoubleEqual(39.370078740157, mm.ToEmu().ToInch().Value);
                AssertDoubleEqual(2834.64566929133, mm.ToPt().Value);
                AssertDoubleEqual(56692.913385826, mm.ToEmu().ToDxa().Value);
                AssertDoubleEqual(36000000, mm.ToEmu().Value);
```

See [Office Open XML file formats - Wikipedia](https://en.wikipedia.org/wiki/Office_Open_XML_file_formats )

[English Metric Units and Open XML](http://polymathprogrammer.com/2009/10/22/english-metric-units-and-open-xml/ )

[Office Open XML Dashboard](http://lcorneliussen.de/raw/dashboards/ooxml/ )

[Points, inches and Emus: Measuring units in Office Open XML – Lars Corneliussen](https://startbigthinksmall.wordpress.com/2010/01/04/points-inches-and-emus-measuring-units-in-office-open-xml/ )

[[译]Points、inches和EMUs：Office Open XML中的度量单位 - 知乎](https://zhuanlan.zhihu.com/p/78307080 )

[Office Open XML 的测量单位](https://blog.lindexi.com/post/Office-Open-XML-%E7%9A%84%E6%B5%8B%E9%87%8F%E5%8D%95%E4%BD%8D.html )





<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。