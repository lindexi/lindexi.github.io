# dotnet 读 WPF 源代码笔记 了解 WPF 已知问题 后台线程创建 WriteableBitmap 锁住主线程

在 WPF 中，如果在没有开启 Dispatcher 的后台线程里面创建 WriteableBitmap 对象，在 WriteableBitmap 构造函数传入在主线程创建的 BitmapSource 也许就会锁住主线程。本文将通过 WPF 框架源代码告诉大家为什么会锁住主线程

<!--more-->
<!-- 发布 -->

这是在 WPF 开源仓库上一个小伙伴报的，详细请看 [WriteableBitmap hangs when source bitmap is rendered on other thread · Issue #4396 · dotnet/wpf](https://github.com/dotnet/wpf/issues/4396 )

复现步骤十分简单，只需要在后台线程创建完成一个 BitmapSource 分别传入给主线程显示和后台线程创建 WriteableBitmap 就会锁住主线程，最简单的代码如下

```csharp
				Task.Run(() =>
				{
					var image = new BitmapImage(new Uri(fileName));

					image.Freeze();  // locks the bitmap source, so other threads can access

					Dispatcher.InvokeAsync(() => Image.Source = (BitmapSource) image);
					//Thread.Sleep(10);   // WPF needs time to render the bitmap. During this period, creating a WriteableBitmap makes the program hang.

					_ = new WriteableBitmap(image);
				});
```

上面代码的 Image 是一个在 XAML 定义的控件

```xml
			<Image x:Name="Image"/>
```

上面的 fileName 是一个文件的路径。详细的测试代码请看 [https://github.com/SetTrend/BitmapSourceTest](https://github.com/lindexi/lindexi_gd/tree/82c1dc09816d7a15214a167cae78f215a3393d6c/BitmapSourceTest)

为什么这个后台线程和主线程会相互等待？原因是在后台线程创建 WriteableBitmap 时，会进入 `WriteableBitmap.InitFromBitmapSource` 方法，在这个方法里面获取了一个主线程后续将会等待的锁。然而后台线程后续需要等待主线程返回，才能完成创建图片，因此主线程在等待后台线程的锁而后台线程在等待主线程返回，两个线程在等待

通过 WPF 仓库的源代码可以看到 `WriteableBitmap.InitFromBitmapSource` 方法的实现如下

```csharp
    public sealed class WriteableBitmap : BitmapSource
    {
        private void InitFromBitmapSource(
            BitmapSource source
            )
        {
        	// Ignore code
        	 _syncObject = source.SyncObject;
            lock (_syncObject)
            {
            	// Ignore code
            }
        }
    }
```

也就是说在后台线程将会拿到创建 WriteableBitmap 构造函数传入的 BitmapSource 的 SyncObject 对象作为锁。对应测试代码的 image 变量的 SyncObject 对象先被后台线程获取，然后在主线程渲染时，也需要用到这个锁，在主线程的堆栈如下

```
 	PresentationCore.dll!System.Windows.Media.Imaging.BitmapSource.UpdateBitmapSourceResource(System.Windows.Media.Composition.DUCE.Channel channel = {System.Windows.Media.Composition.DUCE.Channel}, bool skipOnChannelCheck)
 	PresentationCore.dll!System.Windows.Media.Imaging.BitmapSource.UpdateResource(System.Windows.Media.Composition.DUCE.Channel channel, bool skipOnChannelCheck)
 	PresentationCore.dll!System.Windows.Media.Imaging.BitmapSource.AddRefOnChannelCore(System.Windows.Media.Composition.DUCE.Channel channel = {System.Windows.Media.Composition.DUCE.Channel})
 	PresentationCore.dll!System.Windows.Media.Imaging.BitmapSource.System.Windows.Media.Composition.DUCE.IResource.AddRefOnChannel(System.Windows.Media.Composition.DUCE.Channel channel)
 	PresentationCore.dll!System.Windows.Media.RenderData.System.Windows.Media.Composition.DUCE.IResource.AddRefOnChannel(System.Windows.Media.Composition.DUCE.Channel channel = {System.Windows.Media.Composition.DUCE.Channel})
 	PresentationCore.dll!System.Windows.UIElement.RenderContent(System.Windows.Media.RenderContext ctx, bool isOnChannel)
 	PresentationCore.dll!System.Windows.Media.Visual.UpdateContent(System.Windows.Media.RenderContext ctx = {System.Windows.Media.RenderContext}, System.Windows.Media.VisualProxyFlags flags, bool isOnChannel)
 	PresentationCore.dll!System.Windows.Media.Visual.RenderRecursive(System.Windows.Media.RenderContext ctx = {System.Windows.Media.RenderContext})
```

在主线程渲染图片，需要在 `BitmapSource.UpdateBitmapSourceResource` 方法里面获取锁，请看代码

```csharp
     public abstract class BitmapSource : ImageSource, DUCE.IResource
     {
        internal virtual void UpdateBitmapSourceResource(DUCE.Channel channel, bool skipOnChannelCheck)
        {
                // Ignore code
                // We may end up loading in the bitmap bits so it's necessary to take the sync lock here.
                lock (_syncObject)
                {
                    channel.SendCommandBitmapSource(
                        _duceResource.GetHandle(channel),
                        DUCECompatiblePtr
                        );
                }
            }
        }
     }
```

上面代码的 `_syncObject` 和在后台线程获取的 SyncObject 是相同的对象，因此主线程需要等待后台线程。但是后台线程在执行到 `MediaSystem.Startup` 方法时，就需要等待主线程返回，后台线程调用堆栈如下

```
 	[Manage to Native]	
 	PresentationCore.dll!System.Windows.Media.MediaSystem.Startup(System.Windows.Media.MediaContext mc = {System.Windows.Media.MediaContext})
 	PresentationCore.dll!System.Windows.Media.MediaContext.MediaContext(System.Windows.Threading.Dispatcher dispatcher = {System.Windows.Threading.Dispatcher})
 	PresentationCore.dll!System.Windows.Media.MediaContext.From(System.Windows.Threading.Dispatcher dispatcher)
 	PresentationCore.dll!System.Windows.Media.Imaging.WriteableBitmap.SubscribeToCommittingBatch()
 	PresentationCore.dll!System.Windows.Media.Imaging.WriteableBitmap.Unlock()
 	PresentationCore.dll!System.Windows.Media.Imaging.WriteableBitmap.InitFromBitmapSource(System.Windows.Media.Imaging.BitmapSource source)
 	PresentationCore.dll!System.Windows.Media.Imaging.WriteableBitmap.WriteableBitmap(System.Windows.Media.Imaging.BitmapSource source)
>	BitmapSourceTest.dll!BitmapSourceTest.MainWindow.ProcessImageAsync(string filePath)
 	BitmapSourceTest.dll!BitmapSourceTest.MainWindow.BrowseFile_Click.AnonymousMethod__0()
 	System.Private.CoreLib.dll!System.Threading.Tasks.Task.InnerInvoke()
 	System.Private.CoreLib.dll!System.Threading.Tasks.Task..cctor.AnonymousMethod__277_0(object obj)
 	System.Private.CoreLib.dll!System.Threading.ExecutionContext.RunFromThreadPoolDispatchLoop(System.Threading.Thread threadPoolThread = {System.Threading.Thread}, System.Threading.ExecutionContext executionContext, System.Threading.ContextCallback callback, object state)
 	System.Private.CoreLib.dll!System.Threading.Tasks.Task.ExecuteWithThreadLocal(ref System.Threading.Tasks.Task currentTaskSlot = Id = 189, Status = Running, Method = "Void <BrowseFile_Click>b__0()", System.Threading.Thread threadPoolThread)
 	System.Private.CoreLib.dll!System.Threading.Tasks.Task.ExecuteEntryUnsafe(System.Threading.Thread threadPoolThread)
 	System.Private.CoreLib.dll!System.Threading.Tasks.Task.ExecuteFromThreadPool(System.Threading.Thread threadPoolThread)
 	System.Private.CoreLib.dll!System.Threading.ThreadPoolWorkQueue.Dispatch()
 	System.Private.CoreLib.dll!System.Threading._ThreadPoolWaitCallback.PerformWaitCallback()
```

可以从上面代码看到，主线程在等待后台线程的锁，而后台线程需要等待主线程返回才能释放锁

其实在后台线程创建图片，同时创建的图片的参数还是在主线程使用的图片，这样的逻辑不多，更多使用的是只在后台线程创建图片然后通过 Freeze 给到主线程用来解决性能问题。但上面测试代码的逻辑也不算出错，可以算 WPF 的已知坑。也许我会尝试去修复这个问题

如果不更改 WPF 框架代码，那么一个尝试解决的方法是在后台线程开启 UI 线程，预热一下渲染。预热用来解决后台线程创建 MediaContext 需要等待主线程，通过预先创建，此时可以等待到主线程，如下面代码

```csharp
				Dispatcher backgroundDispatcher = null!;
				AutoResetEvent resetEvent = new AutoResetEvent(false);
				Thread thread = new Thread(() =>
				{
					backgroundDispatcher = Dispatcher.CurrentDispatcher;
					resetEvent.Set();
					Dispatcher.Run();
				});
				thread.SetApartmentState(ApartmentState.STA);
				thread.IsBackground = true;
				thread.Start();

				resetEvent.WaitOne();

				// To Create the MediaContext which is thread static
				backgroundDispatcher.InvokeAsync(() => new WriteableBitmap(1, 1, 96, 96, PixelFormats.Bgr32, null));

				backgroundDispatcher.InvokeAsync(() =>
				{
					var image = new BitmapImage(new Uri(openDialog.FileName));

					image.Freeze(); // locks the bitmap source, so other threads can access

					Dispatcher.InvokeAsync(() => Image.Source = (BitmapSource) image);

					_ = new WriteableBitmap(image);
				});
```

代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/0a8ac8dd9a61599dfa8624f73dfd22fd4b1bc539/BitmapSourceTest) 欢迎小伙伴访问

<!-- 
6713

Why the background task will dead lock? Because the background task will get the lock in `WriteableBitmap.InitFromBitmapSource` method.

```csharp
    public sealed class WriteableBitmap : BitmapSource
    {
        private void InitFromBitmapSource(
            BitmapSource source
            )
        {
        	// Ignore code
        	 _syncObject = source.SyncObject;
            lock (_syncObject)
            {
            	// Ignore code
            }
        }
    }
```

And the `source` is a TransformedBitmap which was created in `ProcessImageAsync` method and running in background task.

```csharp
// The Demo code
		private void ProcessImageAsync(string filePath)
		{
			TransformedBitmap tb = new TransformedBitmap(new BitmapImage(new Uri(filePath)), new RotateTransform(90));

			CopyBitmapSourceToUi(tb);

			_ = new WriteableBitmap(tb);
		}
```

But the `source.SyncObject` will be used in main thread when Render.

```
 	PresentationCore.dll!System.Windows.Media.Imaging.BitmapSource.UpdateBitmapSourceResource(System.Windows.Media.Composition.DUCE.Channel channel = {System.Windows.Media.Composition.DUCE.Channel}, bool skipOnChannelCheck)
 	PresentationCore.dll!System.Windows.Media.Imaging.BitmapSource.UpdateResource(System.Windows.Media.Composition.DUCE.Channel channel, bool skipOnChannelCheck)
 	PresentationCore.dll!System.Windows.Media.Imaging.BitmapSource.AddRefOnChannelCore(System.Windows.Media.Composition.DUCE.Channel channel = {System.Windows.Media.Composition.DUCE.Channel})
 	PresentationCore.dll!System.Windows.Media.Imaging.BitmapSource.System.Windows.Media.Composition.DUCE.IResource.AddRefOnChannel(System.Windows.Media.Composition.DUCE.Channel channel)
 	PresentationCore.dll!System.Windows.Media.RenderData.System.Windows.Media.Composition.DUCE.IResource.AddRefOnChannel(System.Windows.Media.Composition.DUCE.Channel channel = {System.Windows.Media.Composition.DUCE.Channel})
 	PresentationCore.dll!System.Windows.UIElement.RenderContent(System.Windows.Media.RenderContext ctx, bool isOnChannel)
 	PresentationCore.dll!System.Windows.Media.Visual.UpdateContent(System.Windows.Media.RenderContext ctx = {System.Windows.Media.RenderContext}, System.Windows.Media.VisualProxyFlags flags, bool isOnChannel)
 	PresentationCore.dll!System.Windows.Media.Visual.RenderRecursive(System.Windows.Media.RenderContext ctx = {System.Windows.Media.RenderContext})
```

The main thread will use the same SyncObject in BitmapSource.UpdateBitmapSourceResource

```csharp
     public abstract class BitmapSource : ImageSource, DUCE.IResource
     {
        internal virtual void UpdateBitmapSourceResource(DUCE.Channel channel, bool skipOnChannelCheck)
        {
                // Ignore code
                // We may end up loading in the bitmap bits so it's necessary to take the sync lock here.
                lock (_syncObject)
                {
                    channel.SendCommandBitmapSource(
                        _duceResource.GetHandle(channel),
                        DUCECompatiblePtr
                        );
                }
            }
        }
     }
```


The main thread will waitting the `_syncObject` which be used in background task in `WriteableBitmap.InitFromBitmapSource` method.

But the background task now waitting the main thread in `MediaSystem.Startup`. So the main thread wait background task to release the `_syncObject` lock and the background task wait main thread. -->

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、 使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。  
