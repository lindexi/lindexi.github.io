# win10 uwp 自定义标记扩展

在 UWP 使用的 Binding 或 StaticResource 这些都是标记扩展，在 Windows 10 Fall Creators Update 版本号是 10.0.16299.0 和以上支持在 UWP 自定义标记扩展，也就是定义了一个可以在 xaml 使用的标记的方法

<!--more-->
<!-- CreateTime:2019/4/10 9:46:13 -->

<!-- csdn -->

定义一个标记扩展需要满足下面条件

- 继承 [MarkupExtension](https://docs.microsoft.com/en-us/uwp/api/windows.ui.xaml.markup.markupextension?wt.mc_id=MVP) 类
- 重写 ProvideValue 返回值
- 在类上面添加[MarkupExtensionReturnTypeAttribute](https://docs.microsoft.com/en-us/uwp/api/windows.ui.xaml.markup.markupextensionreturntypeattribute?wt.mc_id=MVP) 指定返回的类
- 命名后缀是 Extension 字符串
- 有没有参数的构造函数

下面我简单写一个多语言支持的标记扩展，在界面使用多语言的时候我期望使用这个方式写多语言

```csharp
        <TextBlock Text="{local:Lang Key=lindexi}" />

```

于是我需要创建多语言的类

```csharp
    public class LangExtension : MarkupExtension

```

多语言返回的是字符串，所以标记 [MarkupExtensionReturnTypeAttribute](https://docs.microsoft.com/en-us/uwp/api/windows.ui.xaml.markup.markupextensionreturntypeattribute?wt.mc_id=MVP) 同时设置返回的类

```csharp
    [MarkupExtensionReturnType(ReturnType = typeof(string))]
    public class LangExtension : MarkupExtension
```

添加一个静态字典，用于存放多语言字符串

```csharp
        public static Dictionary<string, string> LangList { set; get; } = new Dictionary<string, string>();

```

添加一个属性，用于绑定的时候输入，从上面代码可以知道我需要一个名为 key 的字符串属性

```csharp
        public string Key { get; set; }

```

重写 ProvideValue 方法，根据用户输入的 Key 返回对应的多语言

```csharp
        protected override object ProvideValue()
        {
            if (LangList.TryGetValue(Key, out var value))
            {
                return value;
            }

            return Key;
        }
```

整个 LangExtension 代码请看

```csharp
    [MarkupExtensionReturnType(ReturnType = typeof(string))]
    public class LangExtension : MarkupExtension
    {
        public string Key { get; set; }

        protected override object ProvideValue()
        {
            if (LangList.TryGetValue(Key, out var value))
            {
                return value;
            }

            return Key;
        }

        public static Dictionary<string, string> LangList { set; get; } = new Dictionary<string, string>();
    }
```

此时就可以在 xaml 使用定义的标记扩展了

```csharp
        <TextBlock Text="{local:LangExtension Key=lindexi}" />
        <TextBlock Text="{local:Lang Key=lindexi}" />
```
 
在使用的时候可以忽略 Extension 字符串

在 [WindowsCommunityToolkit](https://github.com/windows-toolkit/WindowsCommunityToolkit) 也有两个定义，请看 [OnDevice.cs](https://github.com/windows-toolkit/WindowsCommunityToolkit/blob/39858daf1e1b868ae8ac2b0d6f25955b35ff1d81/Microsoft.Toolkit.Uwp.UI/Extensions/Markup/OnDevice.cs ) 和 [NullableBool.cs](https://github.com/windows-toolkit/WindowsCommunityToolkit/blob/3b5a1f480d65c649a1732a8b8a11866ed3c08836/Microsoft.Toolkit.Uwp.UI/Extensions/Markup/NullableBool.cs ) 如果有任何想法欢迎在 [WindowsCommunityToolkit](https://github.com/windows-toolkit/WindowsCommunityToolkit) 讨论

本文使用的源代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/b22ec35cadc8d11c06769e7dcd7aad23750d6cd5/LocerjanayJarberlewerfair) 欢迎评论

[Making the case for XAML Markup Extensions – pedrolamas.com](https://www.pedrolamas.com/2019/03/31/making-the-case-for-xaml-markup-extensions/ )

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。  
