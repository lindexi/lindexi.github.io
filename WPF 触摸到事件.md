# WPF 触摸到事件

本文从代码底层告诉大家，在触摸屏幕之后是如何拿到触摸点并且转换为事件

<!--more-->
<!-- csdn -->
<!-- 草稿 -->

在 WPF 需要使用一个线程去获取触摸的信息，这个线程是在 `PenThreadWorker` 创建，在 `PenThreadWorker` 的构造函数有下面代码

```csharp
			new Thread(new ThreadStart(this.ThreadProc))
			{
				IsBackground = true
			}.Start();
```

通过这个方法就可以创建线程运行 `ThreadProc` 这个函数是一个无限循环，请看代码

```csharp
while (!this.__disposed)
{
	// 忽略代码
}
```

这个函数的底层实际上是包括了另一个循环来从 `penimc2_v0400.dll` 拿到触摸信息。




在拿到触摸信息之后，会调用 `FireEvent` 转换事件，在拿到的信息包括了表示是什么事件

 - 707：PenInRange
 - 708：PenOutOfRange
 - 709：PenDown
 - 710：PenUp
 - 711：Packets

如收到的是 709 事件，就会进入 `FireEvent` 在下面代码使用 `penContext.FirePenDown` 告诉现在是触摸按下

在 FirePenDown 函数会先判断这个触摸是否初始化，每个触摸都有 StylusPointDescription 这个值是使用 `IPimcContext2` 获取 `GetPacketPropertyInfo` 拿到，是触摸屏的设备描述信息里告诉程序这个触摸的精度和触摸宽度

在 `penContext` 传入事件给 `PenContexts.OnPenDown` 这个是在 `PenContexts` 的三个主要事件的一个，可以从代码知道都是调用 `ProcessInput` 只是第一个参数不相同

```csharp
		internal void OnPenDown(PenContext penContext, int tabletDeviceId, int stylusPointerId, int[] data, int timestamp)
		{
			this.ProcessInput(RawStylusActions.Down, penContext, tabletDeviceId, stylusPointerId, data, timestamp);
		}

		internal void OnPenUp(PenContext penContext, int tabletDeviceId, int stylusPointerId, int[] data, int timestamp)
		{
			this.ProcessInput(RawStylusActions.Up, penContext, tabletDeviceId, stylusPointerId, data, timestamp);
		}

		internal void OnPackets(PenContext penContext, int tabletDeviceId, int stylusPointerId, int[] data, int timestamp)
		{
			this.ProcessInput(RawStylusActions.Move, penContext, tabletDeviceId, stylusPointerId, data, timestamp);
		}
```

现在就将事件传入到 ProcessInput 并且告诉 `RawStylusActions` 这个函数会调用 `WispLogic` 的 ProcessInput 在这里使用函数的原因是为了传入的时候加上 `_inputSource` 这里的 `WispLogic` 可能是 StylusLogic 现在的代码就到了比较熟悉的 StylusLogic 函数

在 `WispLogic` 的 `ProcessInput` 会包装输入的参数为 `RawStylusInputReport` 现在这个参数还不需要知道是按下还是移动

```csharp
		internal void ProcessInput(RawStylusActions actions, PenContext penContext, int tabletDeviceId, int stylusDeviceId, int[] data, int timestamp, PresentationSource inputSource)
		{
			RawStylusInputReport inputReport = new RawStylusInputReport(InputMode.Foreground, timestamp, inputSource, penContext, actions, tabletDeviceId, stylusDeviceId, data);
			this.ProcessInputReport(inputReport);
		}
```

这里的第一个参数 `InputMode` 表示的是当前的应用是不是处于前台

然后调用  `WispLogic` 的 `ProcessInput` 传入参数，从这里可以看到，如果一个函数的参数太多，建议创建一个类来传参数。微软在这里做的还是不错

在 `ProcessInput` 需要继续处理参数，使用 StylusDeviceId 找到 `WispStylusDevice` 这个 `WispStylusDevice` 是在 `WispLogic` 的 `RegisterStylusDeviceCore` 注册

在处理完参数会调用 `InvokeStylusPluginCollection` 这个函数会回到 PenContexts 请看代码

```csharp
		internal void InvokeStylusPluginCollection(RawStylusInputReport inputReport)
		{
			if (inputReport.StylusDevice != null)
			{
				inputReport.PenContext.Contexts.InvokeStylusPluginCollection(inputReport);
			}
		}
```

这里需要告诉大家，尽量写比较短代码，将一个功能拆到一个函数，如上面的代码，将多个参数转为一个类就是一个 `ProcessInput` 在添加参数也是在一个函数做。现在需要调用 PenContexts 也是添加一个函数。

看到这里也许会认为 PenContexts 和 PenContext 相同的，实际上是两个不同的类。

```csharp
internal sealed class PenContext
{
		internal PenContext( PenContexts contexts /*忽略参数*/)
		{
			this._contexts = contexts;
			
			// 忽略代码
		}

		internal PenContexts Contexts
		{
			[SecurityCritical]
			get
			{
				return this._contexts;
			}
		}
		private PenContexts _contexts;
}
```

```csharp
internal sealed class PenContexts
{
		private PenContext[] _contexts;

		internal void AddContext(uint index)
		{
			PenContext[] array = _contexts;
			PenContext penContext = this._stylusLogic.TabletDevices[(int)index].As<WispTabletDevice>().CreateContext(this._inputSource.Value.CriticalHandle, this);
			array[(int)index] = penContext;
		}		
}
```

如果想知道 `PenContext` 是如何被创建的，请到下一节，因为这里的代码很多

在 PenContexts 的 `InvokeStylusPluginCollection` 就是调用 StylusPlugIn 的核心代码



 函数 CoalesceAndQueueStylusEvent

## 初始化触摸

初始化触摸需要从 Window 的创建开始说，在 Window 类的 Visibility 修改时触发 `_OnVisibilityChanged` 在这个函数就使用 Dispatcher 调用 ShowHelper 而这个函数就是开始初始化

需要吐槽垃圾微软的函数命名，这里的 `_OnVisibilityChanged` 命名一点都不好，建议方法的命名第一个字符大写，不要使用 `_` 开始。

在 ShowHelper 可以忽略很多代码，因为这里没有明显初始化触摸，初始化的代码是调用 `SafeCreateWindowDuringShow` 调用这个函数可以知道这个函数还不是靠近初始化触摸的代码

中间调用的代码大家可以自己看，这里就直接告诉大家函数调用

 - Window.CreateSourceWindowDuringShow
 - Window.CreateSourceWindow
 - HwndSource.HwndSource
 - HwndSource.Initialize
 - HwndStylusInputProvider.HwndStylusInputProvider
 - WispLogic.RegisterHwndForInput

在 HwndSource.Initialize 的初始化是进行判断，如果设置了 `Switch.System.Windows.Input.Stylus.DisableStylusAndTouchSupport` 就可以根据用户设置打开或不打开，如果是为了学习，建议是打开。这个值默认是 false 也就是打开

然后判断 `StylusLogic.IsPointerStackEnabled` 现在在 dotnet framework 4.7 在 win10 可以使用 Pointer 消息，所以创建的是 `HwndPointerInputProvider` 本文使用的是 `HwndStylusInputProvider` 因为在 Pointer 消息就不用使用本文的逻辑

原来的 WPF 是存在很多的触摸的问题，在 win10 的 UWP 解决了很多的触摸问题的原因是使用了 Pointer 消息。原来的 WPF 是无法收到触摸的消息，需要使用 `penimc2_v0400.dll` 使用一个新的线程去拿到触摸的消息，也就是本文在告诉大家的流程。这个方法存在一些问题，参见[WPF 插拔触摸设备触摸失效](https://lindexi.github.io/lindexi/post/WPF-%E6%8F%92%E6%8B%94%E8%A7%A6%E6%91%B8%E8%AE%BE%E5%A4%87%E8%A7%A6%E6%91%B8%E5%A4%B1%E6%95%88.html )所以建议是使用 Pointer 消息可以解决很多触摸的问题


在 WispLogic.RegisterHwndForInput 就是初始化的函数在开始初始化之前需要 `HwndStylusInputProvider.HwndStylusInputProvider` 初始化 StylusLogic 这个类有一个静态属性


```csharp
	internal abstract class StylusLogic
	{

		internal static StylusLogic CurrentStylusLogic
		{
			get
			{
				if (_currentStylusLogic == null)
				{
					StylusLogic.Initialize();
				}
				
				// 忽略代码

				return _currentStylusLogic;
			}
		}
	}
```

在 `StylusLogic.Initialize` 函数会判断当前是不是使用 Pointer 初始化不同的类

```csharp
       if (StylusLogic.IsPointerStackEnabled)
		{
			StylusLogic._currentStylusLogic = new PointerLogic(InputManager.UnsecureCurrent);
		}
		else
		{
			StylusLogic._currentStylusLogic = new WispLogic(InputManager.UnsecureCurrent);
		}
```

本文不告诉大家 PointerLogic 相关的方法，这里假如是创建 `WispLogic` 就会进入构造，这里只是简单的初始化属性

在 HwndStylusInputProvider.HwndStylusInputProvider 除了创建 StylusLogic 还调用 RegisterHwndForInput 这里传入的是 InputManager HwndSource 通过这两个创建 WispTabletDevices 、 PenContexts 并且通过 IPimcManager2 拿到值

在这个函数调用 `penContexts.Enable()` 就可以创建 PenContext 并且使用，如何创建 PenContext 的过程请看下面

## 创建 PenContext 方法

在触摸的时候很重要的就是 PenContext 在 [WPF 插拔触摸设备触摸失效](https://lindexi.github.io/lindexi/post/WPF-%E6%8F%92%E6%8B%94%E8%A7%A6%E6%91%B8%E8%AE%BE%E5%A4%87%E8%A7%A6%E6%91%B8%E5%A4%B1%E6%95%88.html ) 有告诉了大家，如果拿到 `TabletDeviceInfo` 为空就无法创建 PenContext 那么 这两个有什么联系

创建 PenContext 的核心代码是 WispTabletDevice 类的 CreateContext 创建需要 penContextInfo 而这个值是从 `_penThread.WorkerCreateContext` 拿到，在 WispTabletDevice 这个类可以拿到 PenThread 和 tabletInfo 这两个值都是构造传入

在 WispTabletDeviceCollection 是通过 UpdateTablets 更新 `_tablets` 这个函数在  HwndStylusInputProvider.HwndStylusInputProvider 就已经调用

在窗口打开的过程就通过 HwndStylusInputProvider.HwndStylusInputProvider 调用 `penContexts.Enable()` 创建 PenContext 而 PenContext 的创建需要 PenContextInfo 参数，这个参数需要通过 `_penThread.WorkerCreateContext` 创建

调用 `_penThread.WorkerCreateContext` 不是直接在主线程运行而是在 `PenThreadWorker` 的 Stylus Input 线程运行。

调用 `_penThread.WorkerCreateContext` 时，先在 `_workerOperation` 创建 `WorkerOperationCreateContext` 然后释放 `_pimcResetHandle` 等待 Stylus Input 线程运行。

在 Stylus Input 线程会在 `GetPenEventMultiple` 或 `GetPenEventMultiple`  等待 `_pimcResetHandle` 只有在用户触摸或释放 `_pimcResetHandle` 线程才会继续

<!-- ![](image/WPF 触摸到事件/WPF 触摸到事件0.png) -->

![](http://image.acmx.xyz/lindexi%2F2018810171441731)

这样就创建了 penContext 但是这时还需要将 penContext 加入到 PenThreadWorker 加入的方法是调用 `PenContext.Enable` 通过下面的流程调用 PenThreadWorker.WorkerAddPenContext 添加

```csharp
 	System.Windows.Input.PenThreadWorker.WorkerAddPenContext
 	System.Windows.Input.PenThread.AddPenContext
 	System.Windows.Input.PenThreadPool.GetPenThreadForPenContextHelper
 	System.Windows.Input.PenThreadPool.GetPenThreadForPenContext

```

在 `PenThreadWorker.WorkerAddPenContext` 实际上是添加 `_workerOperation` 的一个值 `WorkerOperationAddContext` 进行添加，也是通过释放 `_pimcResetHandle` 让获得输入的线程运行

在 `WorkerOperationAddContext` 会调用 `PenThreadWorker.AddPenContext` 作为实际添加，通过 `PenThreadWorker.AddPenContext` 会从 PenContext 创建 `_handles` 在触摸线程就是通过判断 `_handles` 数量调用不同的函数

在创建 PenContext 时需要知道创建 PenContext 数量，通过 `WispTabletDeviceCollection.CreateContexts` 遍历从 `UpdateTablets` 更新的 `_tablets` 进行创建。

创建 PenContext 的过程是比较复杂，所以本渣就画了图

<!-- ![](image/WPF 触摸到事件/WPF 触摸到事件1.png) -->

![](http://image.acmx.xyz/lindexi%2F2018810175638248)