# WPF 设置输入只能英文

有时输入只能让用户输入英文，那么如何设置输入只能英文？
<!--more-->
<!-- CreateTime:2019/1/29 15:08:04 -->


<div id="toc"></div>

首先在xaml 写一个 TextBox ，给他一个名字。


```xml
            <TextBox x:Name="txt"></TextBox>
```
然后在构造使用 `System.Windows.Input.InputMethod` 可以设置 IME  和输入是否可以是中文。


```csharp
                System.Windows.Input.InputMethod.SetIsInputMethodEnabled(txt, false);
```

设置IME关掉


```csharp
                InputMethod.SetPreferredImeState(txt,InputMethodState.Off);

```

当然也可以在页面写


```csharp
            <TextBox InputMethod.IsInputMethodEnabled="False"></TextBox>
```


注意用户可以粘贴中文，可以检测用户是否输入有中文。

[C# 切换中英文输入法 - 唐宋元明清2188 - 博客园](https://www.cnblogs.com/kybs0/p/10298697.html )

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。 

