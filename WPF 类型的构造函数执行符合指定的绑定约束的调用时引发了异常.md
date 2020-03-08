# WPF 类型的构造函数执行符合指定的绑定约束的调用时引发了异常

本文告诉大家如果遇到类型“Foo.MainWindow”的构造函数执行符合指定的绑定约束的调用时引发了异常的时候可以如何知道是哪个不清真代码

<!--more-->
<!-- CreateTime:2019/4/12 8:52:35 -->

<!-- csdn -->

在 WPF 开发中，如果遇到类型的构造函数执行符合指定的绑定约束的调用时引发了异常，那么此时通过调用堆栈里面是看不到自己的代码的

```csharp
	PresentationFramework.dll!System.Windows.Markup.WpfXamlLoader.Load(System.Xaml.XamlReader xamlReader, System.Xaml.IXamlObjectWriterFactory writerFactory, bool skipJournaledProperties, object rootObject, System.Xaml.XamlObjectWriterSettings settings, System.Uri baseUri)	
 	PresentationFramework.dll!System.Windows.Markup.WpfXamlLoader.LoadBaml(System.Xaml.XamlReader xamlReader, bool skipJournaledProperties, object rootObject, System.Xaml.Permissions.XamlAccessLevel accessLevel, System.Uri baseUri)	
 	PresentationFramework.dll!System.Windows.Markup.XamlReader.LoadBaml(System.IO.Stream stream, System.Windows.Markup.ParserContext parserContext, object parent, bool closeStream)	
 	PresentationFramework.dll!System.Windows.Application.LoadBamlStreamWithSyncInfo(System.IO.Stream stream, System.Windows.Markup.ParserContext pc)	
 	PresentationFramework.dll!System.Windows.Application.LoadComponent(System.Uri resourceLocator, bool bSkipJournaledProperties)	
 	PresentationFramework.dll!System.Windows.Application.DoStartup()	
 	PresentationFramework.dll!System.Windows.Application..ctor.AnonymousMethod__1_0(object unused)	
 	WindowsBase.dll!System.Windows.Threading.ExceptionWrapper.InternalRealCall(System.Delegate callback, object args, int numArgs)	
 	WindowsBase.dll!System.Windows.Threading.ExceptionWrapper.TryCatchWhen(object source, System.Delegate callback, object args, int numArgs, System.Delegate catchHandler)	
 	WindowsBase.dll!System.Windows.Threading.DispatcherOperation.InvokeImpl()	
 	WindowsBase.dll!System.Windows.Threading.DispatcherOperation.InvokeInSecurityContext(object state)	
 	WindowsBase.dll!MS.Internal.CulturePreservingExecutionContext.CallbackWrapper(object obj)	
 	mscorlib.dll!System.Threading.ExecutionContext.RunInternal(System.Threading.ExecutionContext executionContext, System.Threading.ContextCallback callback, object state, bool preserveSyncCtx)	
 	mscorlib.dll!System.Threading.ExecutionContext.Run(System.Threading.ExecutionContext executionContext, System.Threading.ContextCallback callback, object state, bool preserveSyncCtx)	
 	mscorlib.dll!System.Threading.ExecutionContext.Run(System.Threading.ExecutionContext executionContext, System.Threading.ContextCallback callback, object state)	
 	WindowsBase.dll!MS.Internal.CulturePreservingExecutionContext.Run(MS.Internal.CulturePreservingExecutionContext executionContext, System.Threading.ContextCallback callback, object state)	
 	WindowsBase.dll!System.Windows.Threading.DispatcherOperation.Invoke()	
 	WindowsBase.dll!System.Windows.Threading.Dispatcher.ProcessQueue()	
 	WindowsBase.dll!System.Windows.Threading.Dispatcher.WndProcHook(System.IntPtr hwnd, int msg, System.IntPtr wParam, System.IntPtr lParam, ref bool handled)	
 	WindowsBase.dll!MS.Win32.HwndWrapper.WndProc(System.IntPtr hwnd, int msg, System.IntPtr wParam, System.IntPtr lParam, ref bool handled)	
 	WindowsBase.dll!MS.Win32.HwndSubclass.DispatcherCallbackOperation(object o)	
 	WindowsBase.dll!System.Windows.Threading.ExceptionWrapper.InternalRealCall(System.Delegate callback, object args, int numArgs)	
 	WindowsBase.dll!System.Windows.Threading.ExceptionWrapper.TryCatchWhen(object source, System.Delegate callback, object args, int numArgs, System.Delegate catchHandler)	
 	WindowsBase.dll!System.Windows.Threading.Dispatcher.LegacyInvokeImpl(System.Windows.Threading.DispatcherPriority priority, System.TimeSpan timeout, System.Delegate method, object args, int numArgs)	
 	WindowsBase.dll!MS.Win32.HwndSubclass.SubclassWndProc(System.IntPtr hwnd, int msg, System.IntPtr wParam, System.IntPtr lParam)	
 	[本机到托管的转换]	
 	[托管到本机的转换]	
 	WindowsBase.dll!System.Windows.Threading.Dispatcher.PushFrameImpl(System.Windows.Threading.DispatcherFrame frame)	
 	WindowsBase.dll!System.Windows.Threading.Dispatcher.PushFrame(System.Windows.Threading.DispatcherFrame frame)	
 	PresentationFramework.dll!System.Windows.Application.RunDispatcher(object ignore)	
 	PresentationFramework.dll!System.Windows.Application.RunInternal(System.Windows.Window window)	
 	PresentationFramework.dll!System.Windows.Application.Run(System.Windows.Window window)	
 	PresentationFramework.dll!System.Windows.Application.Run()	
 	CelakercalbochallhiNerjufeeqalchelfu.exe!CelakercalbochallhiNerjufeeqalchelfu.App.Main()	
```

但是此时应该可以找到一些内部异常

很经常可以看到的内部异常有两个

 - “Foo.MainWindow”的类型初始值设定项引发异常。
 - ArgumentException: 默认值类型与属性“Lindexi”类型不匹配。

如果看到是这两个异常，那么请找到默认值类型与属性“Lindexi”类型不匹配里面说到的属性名对应的定义的代码，一般这个属性是依赖属性或附加属性

如我就逗比写了这段代码

```csharp
        public static readonly DependencyProperty LindexiProperty =
            DependencyProperty.Register("Lindexi", typeof(string), typeof(MainWindow), new PropertyMetadata(0));
```

那么上面的代码有什么问题，在依赖属性的定义，需要在 PropertyMetadata 传入的默认参数的类和定义的 `typeof(string)` 是相同的类，如上面代码定义的是字符串，但是在默认值设置的是整数，于是这里就不能转换了。注意，即使隐式转换也是不可以的，如定义的是浮点但是传入整数也是不可以的

解决方法是修改默认值或修改定义的类就可以了

那么为什么在这里定义不对会直接告诉小伙伴是在构造函数绑定的时候炸了？因为定义的是静态字段，在静态字段是会在整个类构造函数之前就执行，于是你就无法在构造函数添加断点找到是哪个不清真代码

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。  
