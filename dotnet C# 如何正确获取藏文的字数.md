# dotnet C# 如何正确获取藏文的字数

在咱国内有很多有趣的文字，其中藏文属于有趣的文字里面特别有趣的一项，特别是对于做文本库的同学，大概都知道什么叫合写字吧。合写字的含义就是多个字符一起组成一个字。但是多个字符在内存中，本身就是多个字符对象，以往统计某个字符串的字数，咱简单判断只是拿字符串的字符数量进行获取。这个方法在藏文下肯定是不可行的，藏文的一个字由多个字符组成，因此需要本文介绍的特别的方法

<!--more-->
<!-- CreateTime:2021/8/23 8:34:19 -->

<!-- 博客 -->

先给大家来一个简单的藏文字 དིོེུ 这个字其实是由 ད + ུ + ི + ོ + ེ 这几个字符组成的

![](http://image.acmx.xyz/lindexi%2F202182285086064.jpg)

用 string.Length 获取到的 དིོེུ 这个字也是符合预期 5 个字符，当然这也是不符合预期的字数

这是关于语言文化方面的内容，自己写一定是不靠谱的。好在 .NET 里面提供的权威的获取方法，通过 StringInfo 类的辅助，可以获取可视效果下的字符串的字数

```csharp
var info = new StringInfo("དིོེུ");
var realLength = info.LengthInTextElements; // realLength = 1
```

通过此即可获取正确的字符长度

额外的，如果想要枚举一个藏文句子的每个藏文的字。那肯定不能使用字符的遍历方式，否则输出就和汉字的遍历输出为偏旁一样了。遍历藏文，需要使用 StringInfo.GetTextElementEnumerator 方法，例子如下

```csharp
var enumerator = StringInfo.GetTextElementEnumerator("ཀྲུང་ཧྭ་མི་དམངས་སྤྱི་མཐུན་རྒྱལ་ཁབ།");
while (enumerator.MoveNext())
{
    Console.WriteLine(enumerator.GetTextElement());
}
```

参阅：

- [2019-11-10-看看藏文里面一共有多少个字吧 - huangtengxiao](https://huangtengxiao.gitee.io/post/%E7%9C%8B%E7%9C%8B%E8%97%8F%E6%96%87%E9%98%BF%E6%8B%89%E4%BC%AF%E6%96%87%E9%87%8C%E9%9D%A2%E4%B8%80%E5%85%B1%E6%9C%89%E5%A4%9A%E5%B0%91%E4%B8%AA%E5%AD%97%E5%90%A7.html)
- [2019-11-10-使用StringInfo正确查找字符个数 - huangtengxiao](https://huangtengxiao.gitee.io/post/%E4%BD%BF%E7%94%A8StringInfo%E6%AD%A3%E7%A1%AE%E6%9F%A5%E6%89%BE%E5%AD%97%E7%AC%A6%E4%B8%AA%E6%95%B0.html)

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。 
