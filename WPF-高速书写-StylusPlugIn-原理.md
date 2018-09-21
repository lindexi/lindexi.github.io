
# WPF 高速书写 StylusPlugIn 原理

本文告诉大家 WPF 的 StylusPlugIn 为什么能做高性能书写，在我的上一篇博客和大家介绍了 WPF 的触摸原理，但是没有详细告诉大家如何通过触摸原理知道如何去做一个高速获得触摸的应用，所以本文就在上一篇博客的基础继续告诉大家底层的原理
如果觉得原理很无论，就直接关闭本文，因为本文都是理论，不会告诉大家如何做高性能书写

<!--more-->


<!-- csdn -->

<!-- 草稿 -->

在 WPF 如果想要做高性能的书写，就需要足够快获得用户的触摸输入，而如果直接拿到的是路由的输入就会存在下面的问题

- 主线程卡住了

- 主线程没有全力处理触摸笔迹

- 路由事件本身的耗时

- 元素多了路由事件就需要经过很多的元素

在用户触摸屏幕的时候，会在 `PenThreadWorker.ThreadProc` 里面的 `UnsafeNativeMethods.GetPenEvent` 或 `UnsafeNativeMethods.GetPenEventMultiple` 拿到触摸的消息，从而调用 `PenThreadWorker.FireEvent` 调用 `PenContext` 的对应的方法

在经过系列的调用最终会调用到 `WispLogic.ProcessInput` 经过封装调用 `WispLogic.ProcessInputReport` 在这个函数里面就会执行所有的 StylusPlugin 请看代码

```csharp
		private void ProcessInputReport(RawStylusInputReport inputReport)
		{
			WispStylusDevice wispStylusDevice = this.FindStylusDeviceWithLock(inputReport.StylusDeviceId);
			inputReport.StylusDevice = ((wispStylusDevice != null) ? wispStylusDevice.StylusDevice : null);
			if (!this._inDragDrop || !inputReport.PenContext.Contexts.IsWindowDisabled)
			{
				this.InvokeStylusPluginCollection(inputReport);
			}
			this.CoalesceAndQueueStylusEvent(inputReport);
		}
```

从上面代码可以看到 `InvokeStylusPluginCollection` 就是运行 StylusPlugin 集合会调用所有的 StylusPlugin 类

那么 WPF 怎么知道当前的程序有哪些 StylusPlugin 可以被调用，下面先从如何寻找开始说

在 `PenContexts.InvokeStylusPluginCollection` 函数会调用 `PenContexts.TargetPlugInCollection` 函数拿到所有可以调用的 `StylusPlugInCollection` 进行运行。

在 `PenContexts.TargetPlugInCollection` 会先尝试拿到已经捕获的 PlugInCollection 进行返回，只有在没有拿到的时候才会执行 `HittestPlugInCollection` 先来看一下 TargetPlugInCollection 的代码，下面的代码被我删除了大部分无关的代码

```csharp
		internal StylusPlugInCollection TargetPlugInCollection(RawStylusInputReport inputReport)
		{
			var wispStylusDevice = inputReport.StylusDevice.As<WispStylusDevice>();
			StylusPlugInCollection stylusPlugInCollection = wispStylusDevice.GetCapturedPlugInCollection(our bool flag);

			// 没有拿到值的时候就使用 HittestPlugInCollection 函数
			if (!flag)
			{
				int[] data = inputReport.Data;
				Point point = new Point((double)data[xx], (double)data[xx]);

				stylusPlugInCollection = this.HittestPlugInCollection(point);
			}
			
			return stylusPlugInCollection;
		}
```

那么 `WispStylusDevice.GetCapturedPlugInCollection` 是如何寻找可以返回的 StylusPlugInCollection 的值，下面请看 GetCapturedPlugInCollection 代码

```csharp
    internal override StylusPlugInCollection GetCapturedPlugInCollection(ref bool elementHasCapture)
    {
        elementHasCapture = _stylusCapture != null;
        return _stylusCapturePlugInCollection;
    }
```

可以看到 GetCapturedPlugInCollection 的代码非常简单，只是返回 `_stylusCapturePlugInCollection` 的值，关于 `_stylusCapturePlugInCollection` 的设置是在  `WispStylusDevice.ChangeStylusCapture` 函数进行设置，至于是什么时候才进行设置，就暂时跳过

这个做法是因为用户可以设置 xx 元素捕获输入，于是无论在哪里按下都需要触发捕获的元素，而忽略了命中到的元素。

一般是无法从 `WispStylusDevice.GetCapturedPlugInCollection` 返回值的，所以就需要使用 `inputReport.Data` 转换点，通过点来做命中测试，找到命中的元素 














<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。