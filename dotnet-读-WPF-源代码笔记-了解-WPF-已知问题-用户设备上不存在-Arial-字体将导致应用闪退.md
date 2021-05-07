
# dotnet 读 WPF 源代码笔记 了解 WPF 已知问题 用户设备上不存在 Arial 字体将导致应用闪退

本文来告诉大家 WPF 已知问题，在用户的设备上，如果不存在 Arial 字体，同时安装了一些诡异的字体，那么也许就会让应用在使用到诡异的字体的时候，软件闪退

<!--more-->


<!-- 标签：WPF，WPF源代码 -->

<!-- 发布 -->

在 WPF 的 FontFamily.cs 字体类里面，有一个叫 FirstFontFamily 的属性，这个属性的逻辑代码里面将包括在当前字体太过诡异时，自动 Fallback 到默认的字体，而默认的字体就是 Arial 字体。这个属性将会在很多逻辑被调用，如获取 FamilyNames 时

```csharp
        public LanguageSpecificStringDictionary FamilyNames
        {
            get
            {
                CompositeFontFamily compositeFont = FirstFontFamily as CompositeFontFamily;
                if (compositeFont != null)
                {
                    // Return the read/write dictionary of family names.
                    return compositeFont.FamilyNames;
                }
                else
                {
                    // Return a wrapper for the cached family's read-only dictionary.
                    return new LanguageSpecificStringDictionary(FirstFontFamily.Names);
                }
            }
        }
```

在进入到寻找 Fallback 字体将会进入到 Invariant 的 Assert 判断方法，在这里面找不到 Arial 字体时，将会进入 Environment.FailFast 让应用程序闪退

以下是 FirstFontFamily 属性的代码，代码我删除了不关键部分

```csharp
 if (family == null) 
 { 
     FontStyle style     = FontStyles.Normal; 
     FontWeight weight   = FontWeights.Normal; 
     FontStretch stretch = FontStretches.Normal; 
     family = FindFirstFontFamilyAndFace(ref style, ref weight, ref stretch); 
  
     if (family == null) 
     { 
     	 // 进入这里的逻辑将会去寻找 Fallback 字体
         // fall back to null font 
         family = LookupFontFamily(NullFontFamilyCanonicalName); 
         Invariant.Assert(family != null); 
     } 
```

在 LookupFontFamily 函数里面，将会尝试去寻找 Arial 字体，上面代码的 NullFontFamilyCanonicalName 默认就是使用 Arial 字体

```csharp
        internal static readonly CanonicalFontFamilyReference NullFontFamilyCanonicalName = CanonicalFontFamilyReference.Create(null, "#ARIAL");
```

在 LookupFontFamily 函数里面将会调用 LookupFontFamilyAndFace 函数去寻找传入的字体，寻找的方法是从 `_defaultFamilyCollection` 去寻找传入的字体

这里的 `_defaultFamilyCollection` 是在静态构造时获取的，代码如下

```csharp
        private static volatile FamilyCollection _defaultFamilyCollection = PreCreateDefaultFamilyCollection();
```

以上的 PreCreateDefaultFamilyCollection 函数，实际就是读取 WindowsFontsUriObject 列表，这里的 Windows 指的不是窗口，而是指 Windows 系统

```csharp
        private static FamilyCollection PreCreateDefaultFamilyCollection()
        {
            FamilyCollection familyCollection = FamilyCollection.FromWindowsFonts(Util.WindowsFontsUriObject);
            return familyCollection;
        }
```

以上的 WindowsFontsUriObject 定义如下

```csharp
        private const string WinDir = "windir";

            string s = Environment.GetEnvironmentVariable(WinDir) + @"\Fonts\";

            _windowsFontsLocalPath = s.ToUpperInvariant();

            _windowsFontsUriObject = new Uri(_windowsFontsLocalPath, UriKind.Absolute);
```

也就是说读取的就是 windir 文件夹下的 Fonts 文件夹，也就是 `C:\Windows\Fonts\` 文件夹

在 LookupFontFamilyAndFace 将会尝试去从 `C:\Windows\Fonts\` 文件夹寻找字体

```csharp
                IFontFamily fontFamily = familyCollection.LookupFamily
                (
                    canonicalFamilyReference.FamilyName,
                    ref style,
                    ref weight,
                    ref stretch
                );
```

假定用户从 `C:\Windows\Fonts\` 文件夹删除了 Arial 字体，那么将找不到字体，返回是空

也就是 LookupFontFamily 将返回空

```csharp
        internal static IFontFamily LookupFontFamily(CanonicalFontFamilyReference canonicalName)
        {
            FontStyle style     = FontStyles.Normal;
            FontWeight weight   = FontWeights.Normal;
            FontStretch stretch = FontStretches.Normal;

            return LookupFontFamilyAndFace(canonicalName, ref style, ref weight, ref stretch);
        }
```

在 FirstFontFamily 属性里面，判断字体存在的代码如下

```csharp
 family = LookupFontFamily(NullFontFamilyCanonicalName);
 Invariant.Assert(family != null); 
```

在用户删除了 Arial 字体，将会让 family 是空，而在 Invariant 的定义代码如下

```csharp
 internal static void Assert(bool condition) 
 { 
     if (!condition) 
     { 
         FailFast(null, null); 
     } 
 } 
```

以上的 FailFast 方法将会调用 Environment.FailFast 方法

```csharp
 private // DO NOT MAKE PUBLIC OR INTERNAL -- See security note 
     static void FailFast(string message, string detailMessage) 
 { 
     if (Invariant.IsDialogOverrideEnabled) 
     { 
         // This is the override for stress and other automation. 
         // Automated systems can't handle a popup-dialog, so let 
         // them jump straight into the debugger. 
         Debugger.Break(); 
     } 
  
     Debug.Assert(false, "Invariant failure: " + message, detailMessage); 
  
     Environment.FailFast(SR.Get(SRID.InvariantFailure)); 
 } 
```

调用 Environment.FailFast 之后，应用程序就闪退了，只有在系统事件里面看到记录

我认为这是一个不合理的设计，至少在框架层不应该有这样的逻辑，作为一个十分成熟的 UI 框架，应该能兼容各个诡异的系统，我将这个问题报告给官方，请看 [WPF known issues: Application will FailFast when not find the Arial font from system · Issue #4464 · dotnet/wpf](https://github.com/dotnet/wpf/issues/4464 )





<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。