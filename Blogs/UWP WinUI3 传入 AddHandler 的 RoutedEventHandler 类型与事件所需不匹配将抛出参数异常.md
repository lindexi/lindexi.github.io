---
title: UWP WinUI3 传入 AddHandler 的 RoutedEventHandler 类型与事件所需不匹配将抛出参数异常
description: 本文记录一个 UWP 或 WinUI3 的开发过程中的问题，当开发者调用 AddHandler 时，所需的 Handler 参数类型为 RoutedEventHandler 类型，然而实际上正确类型是需要与所监听事件匹配才能符合预期工作，否则将抛出缺乏信息的参数异常
tags: UWP,WinUI
category: 
---

<!-- CreateTime:2024/3/2 17:34:41 -->

<!-- 发布 -->
<!-- 博客 -->

开始之前先惯例吐槽一下，我从 2015 开始开发 UWP 应用，然而到 2024 的时候，依然没有看到开发体验上的优化。且在 WinUI3 的技术底层设计上就存在无解问题，那就是许多错误只依靠 COM 的 HR 错误号信息，开发者难以了解真正意义上的调错信息和具体的错误原因。比如说本文所记录的问题

以下是最短复现问题的代码

```csharp
    public MainPage()
    {
        this.InitializeComponent();
        RoutedEventHandler handler = (_, _) =>
        {
            System.Diagnostics.Debug.WriteLine("PointerPressed");
        };

        AddHandler(PointerPressedEvent, handler, true);
    }
```

以上代码是能够通过构建的，原因是 AddHandler 里面的 Handler 参数就是 object 类型的。然而在运行中将会抛出参数异常，异常信息如下

```
System.ArgumentException: Value does not fall within the expected range.
   at WinRT.ExceptionHelpers.<ThrowExceptionForHR>g__Throw|39_0(Int32 hr)
```

异常里面还有 HResult 是 -2147024809 的值。其实这个 -2147024809 需要使用 16 进制去看，结果是有名的 0x80070057 错误号。通过 Error 工具可以看到这表示的是 COM 的通用错误信息，名为 E_INVALIDARG 的错误，意思就是参数错误

```
# for hex 0x80070057 / decimal -2147024809
  COR_E_ARGUMENT                                                 corerror.h
# An argument does not meet the contract of the method.
  DDERR_INVALIDPARAMS                                            ddraw.h
  DIERR_INVALIDPARAM                                             dinput.h
  DSERR_INVALIDPARAM                                             dsound.h
  STIERR_INVALID_PARAM                                           stierr.h
  DRM_E_INVALIDARG                                               windowsplayready.h
  E_INVALIDARG                                                   winerror.h
# One or more arguments are invalid
# as an HRESULT: Severity: FAILURE (1), FACILITY_WIN32 (0x7), Code 0x57
# for hex 0x57 / decimal 87
  ERROR_INVALID_PARAMETER                                        winerror.h
# The parameter is incorrect.
# 8 matches found for "0x80070057"
```

这就是 WinUI3 的一个无解设计问题，通过 HResult 返回错误信息，所包含的信息量太少了，且很多时候距离实际错误点又十分远。应用开发者又不知道 WinUI3 底层投了哪些毒，难以知道所说的参数错误具体指的是什么错误。这一点也是制约了 WinUI 3 的生态，但这一点又是属于 WinUI 3 的基础设计的问题，预估难以更改

这一次的错误信息里面在 Data 里面还包含几条看似没有用，实际也没有用的信息，分别如下

```
+		[0]	{[Description, 不支持此接口
]}	object {System.Collections.DictionaryEntry}
+		[1]	{[RestrictedDescription, 不支持此接口
]}	object {System.Collections.DictionaryEntry}
+		[2]	{[RestrictedErrorReference, ]}	object {System.Collections.DictionaryEntry}
+		[3]	{[RestrictedCapabilitySid, ]}	object {System.Collections.DictionaryEntry}
+		[4]	{[__RestrictedErrorObjectReference, WinRT.ExceptionHelpers+__RestrictedErrorObject]}	object {System.Collections.DictionaryEntry}
+		[5]	{[__HasRestrictedLanguageErrorObject, False]}	object {System.Collections.DictionaryEntry}
```

也就是描述信息里面说的是 `不支持此接口` 的描述信息，合起来就是：遇到参数错误了，因为底层不支持参数传进来的此接口

但是就是不告诉大家，具体错误的是哪个参数，且错在哪里了。要是能够明白说明 handler 参数的类型不符合预期之类的，那开发者的调试效率将会高出许多

本文记录的错误问题原因是 PointerPressedEvent 所对应的是 PointerEventHandler 类型，而不是 RoutedEventHandler 类型，修复的代码如下

```csharp
        PointerEventHandler handler = (_, _) =>
        {
            System.Diagnostics.Debug.WriteLine("PointerPressed");
        };

        AddHandler(PointerPressedEvent, handler, true);
```

那日常开发过程中，如何知道 AddHandler 里面的 handler 参数应该传入什么类型的委托呢？其实方法很简单，只需要使用对应的事件，看看对应的事件定义是什么。比如 PointerPressedEvent 对应的就是 PointerPressed 事件，按照通用命名法就是对应的事件就是对应路由事件定义去掉 Event 后缀。通过查阅文档或者是在 VisualStudio 里面点点看，就可以看到对应的事件的定义，如下面代码就是 PointerPressed 的定义，可以看到事件是 PointerEventHandler 类型的委托

```csharp
public event PointerEventHandler PointerPressed { add; remove; }
```

通过此方式即可知道传入 AddHandler 的 handler 应该使用什么样的类型，解决运行时失败的原因。常见的错误都在于更改代码的时候，忘记同步更改对应的委托类型

额外补充一点，以上的代码的 handler 局部变量是安全的，不会被回收，原因是虽然在以上代码里面看起来 handler 局部变量没被引用，然而在 AddHandler 底层里面已经做好了引用，不会导致 handler 被回收，从而导致 COM 层访问被回收的内存而炸掉的问题。但是此问题在古老的 UWP 是存在的。一个推荐的优化方法就是将 handler 存放在字段里面，手动防止被回收

本文代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/d43a62536b449ef337160f9931265a0db482ed12/FelawchechadaGeqedaihallnela) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/d43a62536b449ef337160f9931265a0db482ed12/FelawchechadaGeqedaihallnela) 上，可以使用如下命令行拉取代码

先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin d43a62536b449ef337160f9931265a0db482ed12
```

以上使用的是 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码，将 gitee 源换成 github 源进行拉取代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin d43a62536b449ef337160f9931265a0db482ed12
```

获取代码之后，进入 FelawchechadaGeqedaihallnela 文件夹，即可获取到源代码
