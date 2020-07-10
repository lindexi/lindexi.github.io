
# dotnet OpenXML 文本字体的选择规则

在 Office 的文本排版里面，会根据字符选择使用哪个字体插槽。也就是实际上在 Office 里面可以在一个文本段里面指定多个字体，会根据实际的字符使用不同的字体

<!--more-->


<!-- CreateTime:7/8/2020 4:40:01 PM -->

<!-- 发布 -->

在做 Office 解析的时候，在 OpenXML SDK 里面是没有找到表示字体的属性的，只能找到 LatinFont 和 EastAsianFont 和 ComplexScriptFont 和 SymbolFont 这几个

```csharp
        public TextFontType LatinFont { get; set; } // latin
        public TextFontType EastAsianFont { get; set; } // ea
        public TextFontType ComplexScriptFont { get; set; } // cs
        public TextFontType SymbolFont { get; set; } // sym
```

而这每个里面都可以使用不同的字体，如下面的文档

```xml
<a:r>
   <a:rPr …>
     <a:cs typeface="Times New Roman"/>
     <a:latin typeface="songti"/>
   </a:rPr>
   <a:t>
 العربية 
   </a:t> 
</a:r>
```

可以看到这里包含了 `a:cs` 和 `a:latin` 分别表示 LatinFont 和 ComplexScriptFont 两个不同的插槽

那么具体的这个我也看不懂的文本应该使用 `Times New Roman` 还是使用宋体字体，这就需要使用下面这个表格的内容了

在 Office 里面将会根据字符的 Unicode 决定使用哪个字体插槽的字体

- U+0000–U+007F latin font 
- U+0080–U+00A6 latin font 
- U+00A9–U+00AF latin font 
- U+00B2–U+00B3 latin font 
- U+00B5–U+00D6 latin font 
- U+00D8–U+00F6 latin font 
- U+00F8–U+058F latin font 
- U+0590–U+074F cs font 
- U+0780–U+07BF cs font 
- U+0900–U+109F cs font 
- U+10A0–U+10FF latin font 
- U+1200–U+137F latin font 
- U+13A0–U+177F latin font 
- U+1D00–U+1D7F latin font 
- U+1E00–U+1FFF latin font 
- U+1780–U+18AF cs font 
- U+2000–U+200B latin font 
- U+200C–U+200F cs font 
- U+2010–U+2029 latin font
- U+2018–U+201F ea font
- U+202A–U+202F cs font 
- U+2030–U+2046 latin font 
- U+204A–U+245F latin font 
- U+2670–U+2671 cs font 
- U+27C0–U+2BFF latin font 
- U+3099–U+309A ea font  
- U+D835        latin font
- U+F000–U+F0FF sym font 
- U+FB00–U+FB17 latin font 
- U+FB1D–U+FB4F cs font 
- U+FE50–U+FE6F latin font
- U+1D400–U+1D7FF latin font 
- 其他 ea font

<!-- (U\+[\dA-F]*\SU\+[\dA-F]*) -->

因此这和 [C# dotnet 获取某个字符所在 Unicode 字符平面映射](https://blog.lindexi.com/post/C-dotnet-%E8%8E%B7%E5%8F%96%E6%9F%90%E4%B8%AA%E5%AD%97%E7%AC%A6%E6%89%80%E5%9C%A8-Unicode-%E5%AD%97%E7%AC%A6%E5%B9%B3%E9%9D%A2%E6%98%A0%E5%B0%84.html) 不相同

这是 [ECMA-376](http://www.ecma-international.org/publications/standards/Ecma-376.htm ) 规定的





<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。