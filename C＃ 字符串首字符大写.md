# C＃ 字符串首字符大写

我找到一些把字符串首字符大写的方法。

<!--more-->
<!-- CreateTime:2020/1/7 11:39:44 -->


假如需要把字符串 "red" 转换为 "Red"，把 "red house" 转为 "Red house" 或者单词的第一个大写，下面就是我从网上看到的技术。

```csharp
public static string FirstCharToUpper(string input)
{
    if (string.IsNullOrEmpty(input))
        throw new ArgumentException("ARGH!");
    return input.First().ToString().ToUpper() + input.Substring(1);
}
```

这个方法就是拿到第一个字符，然后加上后面的字符，可以看到这个方法需要三个字符串在内存。需要解释一下的就是为什么明明看到只有两个字符串变量怎么会有三个字符串？实际上这里的第一个字符串是 `input.First().ToString()` ，
第二个字符串是 `input.First().ToString().ToUpper()` ，ToUpper 就是会创建一个字符串。
第三个字符串是 `input.Substring(1)`，这几个字符串在方法运行完就会被去掉。如果这个方法执行次数很多，那么就需要不停清除字符串。

```csharp
public string FirstLetterToUpper(string str)
{
    if (str == null)
        return null;

    if (str.Length > 1)
        return char.ToUpper(str[0]) + str.Substring(1);

    return str.ToUpper();
}
```

这个方法也是需要三个字符串。

下面的方法大概大家比较少会去发现，就是 CultureInfo 的方法

```csharp
CultureInfo.CurrentCulture.TextInfo.ToTitleCase(str.ToLower());
```

这个方法是比较好方法，假如我输入"red house" 那么就会转换为 "Red House"

上面的方法还可以使用这个方法

```csharp
CultureInfo("en-US").TextInfo.ToTitleCase("red house");
```


如果需要使用拼接，可以使用这个方法

```csharp
s.Remove(1).ToUpper() + s.Substring(1) 
```

上面这个方法不会把 "red house" 转换为 "Red House"

下面给大家一个性能比较好的方法

```csharp
        char[] a = s.ToCharArray();
        a[0] = char.ToUpper(a[0]);
        return new string(a);
```

如果需要很多字符串都这样把第一个大写，可以使用下面方法

```csharp
string str = "red house";
            Console.WriteLine(System.Text.RegularExpressions.Regex.Replace(str, "^[a-z]", m => m.Value.ToUpper()));
```

和上面方法一样写法，可以使用另外的函数

```csharp
Regex.Replace(str, @"^\w", t => t.Value.ToUpper());
```

如果希望有最好的速度，那么请用下面方法

```csharp
public static unsafe string ToUpperFirst(this string str)
{
    if (str == null) return null;
    string ret = string.Copy(str);
    fixed (char* ptr = ret) 
        *ptr = char.ToUpper(*ptr);
    return ret;
}
```

![](http://image.acmx.xyz/34fdad35-5dfe-a75b-2b4b-8c5e313038e2%2F2017917102022.jpg)

我认为，在字符串大写这个算法，不需要去找性能最好的，需要找的是最容易让别人看懂的才是最好的。

https://stackoverflow.com/q/4135317/6116637

## 感谢

[~雨落忧伤~ - 博客园](http://www.cnblogs.com/cjm123/ ) 大神在博客园告诉我文章里写不清晰的内容

本文章同时发在博客园 [C＃ 字符串首字符大写 ](http://www.cnblogs.com/lindexi/p/CFirstCharToUpper.html )

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。