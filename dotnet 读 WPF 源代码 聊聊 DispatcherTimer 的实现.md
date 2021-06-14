# dotnet 读 WPF 源代码 聊聊 DispatcherTimer 的实现

本文来告诉大家在 WPF 框架里面，是如何实现 DispatcherTimer 的功能。有小伙伴告诉我，读源代码系列的博客看不动，原因是太底层了。我尝试换一个方式切入逻辑，通过提问题和解决问题的方法，一步步告诉大家 WPF 是如何实现 DispatcherTimer 的功能

<!--more-->
<!-- CreateTime:2021/6/7 20:44:13 -->

<!-- 发布 -->

假定咱是 WPF 框架的开发者（虽然我就是，尽管是格式化代码工程师）咱需要实现一个 DispatcherTimer 的功能，请问可以如何写呢

在 Windows 上有很多方式来实现计时器的功能，但是 DispatcherTimer 和其他的计时器有一点不同的在于，毕竟这是 Dispatcher 的，看到 Dispatcher 就可以了解到，这是一个需要在主线程执行的定时器

在那么如何在定时器里面回到主线程呢？假定咱现在啥都没有，毕竟咱现在是在从零开发 WPF 框架的，那有什么可以使用呢？在 Windows 上提供了 [SetTimer](https://docs.microsoft.com/en-us/windows/win32/api/winuser/nf-winuser-settimer?WT.mc_id=WD-MVP-5003260 ) 这个放在 User32.dll 的函数，通过这个 Win32 方法可以调用 Windows 提供的底层定时器的功能

写过 Win32 代码的小伙伴就知道，如果直接使用 Win32 的方法，无论是参数还是需要了解的知识都是非常多的。作为一个有追求的框架，咱肯定是需要再做一层封装，让调用更加简单。回到 [SetTimer](https://docs.microsoft.com/en-us/windows/win32/api/winuser/nf-winuser-settimer?WT.mc_id=WD-MVP-5003260 ) 这个 Win32 函数的功能上，咱可以调用 [SetTimer](https://docs.microsoft.com/en-us/windows/win32/api/winuser/nf-winuser-settimer?WT.mc_id=WD-MVP-5003260 ) 给定一个窗口句柄以及计时的时间，接下来 Windows 将会定时发送 [`WM_Timer`](https://docs.microsoft.com/en-us/windows/win32/winmsg/wm-timer?WT.mc_id=WD-MVP-5003260) 给到咱的窗口

假定咱已经有了接收窗口消息的统一入口，接受窗口调度的模块的功能就是调度执行，也就是 Dispatcher 的一个功能。那不妨就将 [`WM_Timer`](https://docs.microsoft.com/en-us/windows/win32/winmsg/wm-timer?WT.mc_id=WD-MVP-5003260) 的处理也放在 Dispatcher 里面吧。刚好咱选用的 [SetTimer](https://docs.microsoft.com/en-us/windows/win32/api/winuser/nf-winuser-settimer?WT.mc_id=WD-MVP-5003260 ) 是发送窗口消息，自然就是被主线程收到了，咱也就不需要去尝试解决后台线程的计时器需要调度到主线程

对于上层的 API 封装呢？给开发者使用的计时器肯定是需要封装一个类，那就叫 DispatcherTimer 好了。至于 DispatcherTimer 里面有哪些 API 呢，就抄 WPF 的设计好了

这里有一个问题是，假定我使用的是 DispatcherTimer 有多个，我使用其中的一个 DispatcherTimer 通过 [SetTimer](https://docs.microsoft.com/en-us/windows/win32/api/winuser/nf-winuser-settimer?WT.mc_id=WD-MVP-5003260 ) 这个 Win32 函数进行定时，在 Dispatcher 收到 [`WM_Timer`](https://docs.microsoft.com/en-us/windows/win32/winmsg/wm-timer?WT.mc_id=WD-MVP-5003260) 消息时，如果知道是需要调用哪个 DispatcherTimer 来执行？

通过分析需求，事实上这个问题不好解决，因为 Win32 的 [`WM_Timer`](https://docs.microsoft.com/en-us/windows/win32/winmsg/wm-timer?WT.mc_id=WD-MVP-5003260) 消息是不会告诉咱这个消息是被哪个逻辑调用的 [SetTimer](https://docs.microsoft.com/en-us/windows/win32/api/winuser/nf-winuser-settimer?WT.mc_id=WD-MVP-5003260 ) 方法调用的，不能通过 `WM_Timer` 获取 DispatcherTimer 对象

但是从需求分析，其实咱不需要关注收到消息对应的是哪个 DispatcherTimer 对象，因为 DispatcherTimer 对象的功能是执行 Tick 事件，而只要是时间刚好到达，就需要执行 Tick 事件了。为了实现此功能，咱也就需要有一个集合用来管理当前主线程所有的 DispatcherTimer 对象，用来了解在收到 `WM_Timer` 需要调用的 DispatcherTimer 对象有哪些

这个 DispatcherTimer 集合为了方便调用管理，不妨先放在 Dispatcher 类里面，毕竟一个线程就刚好有一个 Dispatcher 对象

```csharp
    public sealed class Dispatcher
    {
        private List<DispatcherTimer> _timers = new List<DispatcherTimer>();

        internal void AddTimer(DispatcherTimer timer)
        {
            lock(_instanceLock)
            {
               // 忽略代码
               _timers.Add(timer);
            }

            // 忽略代码
        }
    }
```

那在啥时候需要调用 AddTimer 呢？在 DispatcherTimer 对象创建的时候？如果我只是创建一个空的 DispatcherTimer 对象，这个对象啥都不干，好像加入到 Dispatcher 的 `_timers` 也不合适。不如就在 DispatcherTimer 启动的时候添加

```csharp
    public class DispatcherTimer
    {
        public void Start()
        {
            lock(_instanceLock)
            {
                if(!_isEnabled)
                {
                    _isEnabled = true;

                    _dispatcher.AddTimer(this);
                }
            }
        }

        private Dispatcher _dispatcher;
        private bool _isEnabled;

        // 忽略代码
    }
```

在收到 `WM_Timer` 事件，就需要 Dispatcher 去遍历所有的 DispatcherTimer 对象，看哪个对象当前需要被执行了。为了了解哪个 DispatcherTimer 需要被执行，就需要让 DispatcherTimer 记录两个信息，一个是距离下次执行的时间和调用执行 Start 函数的时间。通过判断调用 Start 的时间加上距离下次执行的时间是否小于或等于当前的时间，就可以判断当前的 DispatcherTimer 是否需要执行

咱来加一点代码在 DispatcherTimer 里面，在启动时记录时间

```csharp
        public void Start()
        {
            lock(_instanceLock)
            {
                if(!_isEnabled)
                {
                    _isEnabled = true;

                    _dispatcher.AddTimer(this);

                    _startTime = DateTime.Now;
                }
            }
        }

        private DateTime _startTime;
        private TimeSpan _interval;
```

作为一个追求性能的框架，自然咱需要在每个地方都追求一下性能，例如获取当前时间，是否有更快的方法？通过 Environment.TickCount 属性可以获取更快的时间，使用 Environment.TickCount 获取的是毫秒数，表示的是开机到当前的时间，相对来说抽象一点，不过也刚好不会受到用户修改当前系统时间的影响，自然也就更稳定一些啦

既然都使用 Environment.TickCount 了，不如将 判断调用 Start 的时间加上距离下次执行的时间 合在一起计算吧，这样后续每次 `WM_Timer` 消息过来的时候，就不用每次都做一次加法了，直接判断值的大小即可

```csharp
        public void Start()
        {
            lock(_instanceLock)
            {
                if(!_isEnabled)
                {
                    _isEnabled = true;

                    _dispatcher.AddTimer(this);

                    // 如果只是记录当前调用 Start 方法的时间，也就是 Environment.TickCount 时间。那么后续收到 WM_Timer 消息，都需要判断当前时间加上 _interval 的时间之后是否小于等于当前的时间。而这个加法计算是每次都需要调用的，为了性能优化，不如一开始就加上，后续就只需要判断大小
                    _dueTimeInTicks = Environment.TickCount + (int) _interval.TotalMilliseconds;
                }
            }
        }

        // 删除 DateTime 的定义，因为获取的性能不够，而且用户也许修改系统时间
        // private DateTime _startTime;
        private TimeSpan _interval;

        // 用这个代替 DateTime 的方法，单位是毫秒。其实字段从规范来说是不应该 internal 公开的，然而在 WPF 里面，古老的开发者为了减少改动就公开了这个字段
        internal int _dueTimeInTicks; // used by Dispatcher
```

在 Dispatcher 里面就可以通过 DispatcherTimer 的 `_dueTimeInTicks` 字段和当前的时间比较大小而决定是否触发 DispatcherTimer 的事件。从规范的角度来说，是不能公开 DispatcherTimer 的 `_dueTimeInTicks` 字段的，然而在 WPF 里面，古老的开发者为了减少改动就公开了这个字段

在 Dispatcher 里面的代码如下

```csharp
    public sealed class Dispatcher
    {
        private List<DispatcherTimer> _timers = new List<DispatcherTimer>();

        private IntPtr WndProcHook(IntPtr hwnd, int msg, IntPtr wParam, IntPtr lParam, ref bool handled)
        {
        	WindowMessage message = (WindowMessage) msg;

            // 忽略代码
        	if(message == WindowMessage.WM_TIMER && (int) wParam == TIMERID_TIMERS)
            {
            	// 忽略代码
                PromoteTimers(Environment.TickCount);
            }
        }

        internal void PromoteTimers(int currentTimeInTicks)
        {
                    DispatcherTimer timer = null;
                    int iTimer = 0;
                    var timersVersion = _timersVersion;

                    do
                    {
                        lock(_instanceLock)
                        {
                            timer = null;

                            // If the timers collection changed while we are in the middle of
                            // looking for timers, start over.
                            if(timersVersion != _timersVersion)
                            {
                            	// 如果在循环过程，有其他逻辑加入了 _timers 的元素，意味着 _timers 的数量变更了
                            	// 需要重新开始
                                timersVersion = _timersVersion;
                                iTimer = 0;
                            }

                            while(iTimer < _timers.Count)
                            {
                                // WARNING: this is vulnerable to wrapping
                                if(_timers[iTimer]._dueTimeInTicks - currentTimeInTicks <= 0)
                                {
                                    timer = _timers[iTimer];

                                    // 忽略代码
                                    break;
                                }
                                else
                                {
                                    iTimer++;
                                }
                            }
                        }

                        // Now that we are outside of the lock, promote the timer.
                        if(timer != null)
                        {
                            timer.Promote();
                        }
                    } while(timer != null);
        }
    }
```

以上判断是通过 `_timers[iTimer]._dueTimeInTicks - currentTimeInTicks <= 0` 决定是否当前的 Timer 需要执行。因为 `_timers[iTimer]._dueTimeInTicks - currentTimeInTicks <= 0` 等价于 `_timers[iTimer]._dueTimeInTicks <= currentTimeInTicks` 也就是在 DispatcherTimer 下次执行的时间，小于或等于当前的时间，这个 DispatcherTimer 就应该被执行。因为相同的时间需要执行的 DispatcherTimer 也许有多个，因此就做了两重循环。而同时为了解决在 DispatcherTimer 执行过程，也许有其他逻辑再加入新的 DispatcherTimer 因此也就需要判断一下 `_timersVersion` 当前版本适合和进入的版本相同，如果不同，就证明有其他逻辑更改了集合，需要重新开始

从上面代码可以看到，咱判断 DispatcherTimer 是否需要被执行，如果需要执行，调用 DispatcherTimer 的 Promote 方法进行执行，最简单的方法执行就是通过调用 Tick 事件触发，简单的代码如下

```csharp
        private void FireTick()
        {
            // 忽略代码
            if(Tick != null)
            {
                Tick(this, EventArgs.Empty);
            }
        }

        internal void Promote() // called from Dispatcher
        {
            FireTick();
        }
```

既然所有的 DispatcherTimer 都被 Dispatcher 放在一起，那是否可以共用一个 Win32 的计时器，不需要每个 DispatcherTimer 都独立调用。如上面的代码，其实都是在判断统一的时间，不需要多个 Win32 计时器也能实现效果

只需要有一个 Win32 计时器，定时是当前的 DispatcherTimer 里面最短的时间，就可以实现多个 DispatcherTimer 使用相同的一个 Win32 计时器。那这个逻辑可以放在哪呢？是否还记得咱在启动计时器时加入到 Dispatcher 里面，既然咱期望多个 DispatcherTimer 使用相同的一个 Win32 计时器，不妨找到一对多的关系，刚好这里的一就是 Dispatcher 类，这里的多就是 DispatcherTimer 类。 因此这个 Win32 计时器的管理，放在 Dispatcher 里面就刚好。启动或者重新设置 Win32 计时器可以放在 Dispatcher 的 AddTimer 方法里面

```csharp

    public sealed class Dispatcher
    {
        private List<DispatcherTimer> _timers = new List<DispatcherTimer>();
        private long _timersVersion;

        internal void AddTimer(DispatcherTimer timer)
        {
            lock(_instanceLock)
            {
                _timers.Add(timer);
                _timersVersion++;
            }
            UpdateWin32Timer();
        }
    }
```

在加入 AddTimer 调用 UpdateWin32Timer 更新计时器时间，原因是如果我原有一个是定时是 10 秒的计时器在启动了。接下来运行了 5 秒，我再加入一个需要等 1 秒的计时器，那么原有的 Win32 计时器是不是就需要更新一下时间？从原来的等待 10 秒，判断距离现在还有 5 秒才执行，而新加入的等待 1 秒的计时器，在接下来的 1 秒就需要执行，那么就需要更新 Win32 计时器，修改定时时间

而如果原有一个是定时是 10 秒的计时器在启动了。接下来运行了 9 秒，我再加入一个需要等 3 秒的计时器，显然新加入的计时器还需要等待 3 秒才执行，而原有的计时器，只需要再等待 1 秒就足够 10 秒了，可以执行。此时的 Win32 计时器自然是不需要重新启动的

似乎上面的逻辑稍微有一点绕，但是看起来代码也是很简单的

```csharp
    public sealed class Dispatcher
    {
        private int _dueTimeInTicks;
    	private bool _dueTimeFound;

        internal void UpdateWin32Timer() // Called from DispatcherTimer
        {
                    bool oldDueTimeFound = _dueTimeFound;
                    int oldDueTimeInTicks = _dueTimeInTicks;
                    _dueTimeFound = false;
                    _dueTimeInTicks = 0;

                    if(_timers.Count > 0)
                    {
                        // We could do better if we sorted the list of timers.
                        for(int i = 0; i < _timers.Count; i++)
                        {
                            DispatcherTimer timer = _timers[i];

                            if(!_dueTimeFound || timer._dueTimeInTicks - _dueTimeInTicks < 0)
                            {
                                _dueTimeFound = true;
                                _dueTimeInTicks = timer._dueTimeInTicks;
                            }
                        }
                    }

                    if(_dueTimeFound)
                    {
                        if(!_isWin32TimerSet || !oldDueTimeFound || (oldDueTimeInTicks != _dueTimeInTicks))
                        {
                            SetWin32Timer(_dueTimeInTicks);
                        }
                    }
                    else if(oldDueTimeFound)
                    {
                        KillWin32Timer();
                    }
        }
    }
```

大概这样就算完成了 DispatcherTimer 的核心实现了，不过此时让咱去天台将性能优化组救下。性能优化组说如果有连续的多个 DispatcherTimer 在执行，此时界面上就卡不动了。因为咱上面的代码，多个 DispatcherTimer 执行之间是没有切换调度的，也就是说刚好有多个 DispatcherTimer 都在执行，那么主线程的资源都在去处理其他业务逻辑里，没有资源去处理界面渲染等

产品大佬也加了需求，要求在 DispatcherTimer 可以加入优先级，优先级相等于 Dispatcher 的优先级，于是咱的逻辑代码也需要改改

在 DispatcherTimer 的 Promote 方法里面，看起来不能调用 FireTick 开始执行代码逻辑，而是需要有优先级调度，也需要有切换调度，不能将全部的 DispatcherTimer 一次性执行。最简单的方法自然就是 Dispatcher.InvokeAsync 等方法来实现优先级调度等功能

产品大佬的需求实现了，但性能优化组还在天台上，咱还需要再优化一下。既然都将 DispatcherTimer 加入到 Dispatcher 里面了，那为什么还需要 Dispatcher.InvokeAsync 调度呢？最简单的方法就是在 DispatcherTimer 启动的时候，将任务加入到 Dispatcher 里面，但是设置优先级为不执行。当 DispatcherTimer 的 Promote 调用时，设置刚才的加入的任务的优先级为 DispatcherTimer 的执行优先级，自然就会被 Dispatcher 进行调度了

```csharp
    public class DispatcherTimer
    {
        public DispatcherTimer(DispatcherPriority priority) // NOTE: should be Priority
        {
            _priority = priority;
        }

        private void Start()
        {
            lock(_instanceLock)
            {
                if (_operation != null)
                {
                    // Timer has already been restarted, e.g. Start was called form the Tick handler.
                    return;
                }

                // BeginInvoke a new operation.
                _operation = _dispatcher.BeginInvoke(
                    DispatcherPriority.Inactive,
                    new DispatcherOperationCallback(FireTick),
                    null);

                
                _dueTimeInTicks = Environment.TickCount + (int) _interval.TotalMilliseconds;
                
                _dispatcher.AddTimer(this);
            }
} // 这确实是 WPF 的格式化，这是花括号前面没有空格

        internal void Promote() // called from Dispatcher
        {
            lock(_instanceLock)
            {
                // Simply promote the operation to it's desired priority.
                if(_operation != null)
                {
                    _operation.Priority = _priority;
                }
            }
        }

        private object FireTick(object unused)
        {
            if(Tick != null)
            {
                Tick(this, EventArgs.Empty);
            }

            return null;
        }

        private DispatcherPriority _priority;  // NOTE: should be Priority
    }
```

通过上面的代码，性能优化组从天台上下来了，但产品大佬又说，有一些用户喜欢在 Tick 里面里面将 DispatcherTimer 停下，而以上的代码，其实咱没有实现停下的功能，刚好两个功能一起做

在 DispatcherTimer 里面定义 IsEnabled 属性，咱需要支持在 IsEnabled 里面进行赋值从而进行停止或启动计时器

```csharp
    public class DispatcherTimer
    {
        /// <summary>
        ///     Gets or sets whether the timer is running.
        /// </summary>
        public bool IsEnabled
        {
            get
            {
                return _isEnabled;
            }

            set
            {
                lock(_instanceLock)
                {
                    if(!value && _isEnabled)
                    {
                        Stop();
                    }
                    else if(value && !_isEnabled)
                    {
                        Start();
                    }
                }
            }
        }

        private bool _isEnabled;
    }
```

既然有不断的开启和停止，那不如就再加一个 Restart 方法好了，让 Start 方法调用 Restart 方法

```csharp
    public class DispatcherTimer
    {
        public void Start()
        {
            lock(_instanceLock)
            {
                if(!_isEnabled)
                {
                    _isEnabled = true;

                    Restart();
                }
            }
        }

        private void Restart()
        {
            lock(_instanceLock)
            {
                if (_operation != null)
                {
                    // Timer has already been restarted, e.g. Start was called form the Tick handler.
                    return;
                }

                // BeginInvoke a new operation.
                _operation = _dispatcher.BeginInvoke(
                    DispatcherPriority.Inactive,
                    new DispatcherOperationCallback(FireTick),
                    null);

                
                _dueTimeInTicks = Environment.TickCount + (int) _interval.TotalMilliseconds;
                
                _dispatcher.AddTimer(this);
            }
}
    }
```

那 Stop 方法呢？其实就是从 Dispatcher 队列里面干掉 `_operation` 对象

```csharp
    public class DispatcherTimer
    {
        public void Stop()
        {
              if(_operation != null)
              {
                   _operation.Abort();
                   _operation = null;
              }
        }
    }
```

当然了，如果当前计时最短就是当前的被 Stop 的 DispatcherTimer 那还需要更新一下 Win32 的计时器时间。例如当前已设置了最短的计时是 1 秒的 DispatcherTimer 被 Stop 了，而后续的  DispatcherTimer 是再等 5 秒，此时就需要修改 Win32 的计时器，关闭等待 1 秒的计时器，再开启等待 5 秒的计时器。另外咱将 DispatcherTimer 加入到 Dispatcher 的一个集合里面，自然就需要在 Stop 里面移除，否则将会让 DispatcherTimer 对象无法释放

咱更改 Stop 方法，加上告诉 Dispatcher 的方法

```csharp
    public class DispatcherTimer
    {
        public void Stop()
        {
            bool updateWin32Timer = false;
            
            lock(_instanceLock)
            {
                if(_isEnabled)
                {
                    _isEnabled = false;
                    updateWin32Timer = true;

                    // If the operation is in the queue, abort it.
                    if(_operation != null)
                    {
                        _operation.Abort();
                        _operation = null;
                    }
}
            }

            if(updateWin32Timer)
            {
                _dispatcher.RemoveTimer(this);
            }
        }
    }
```

在 Dispatcher 的里面，先从集合里面将 DispatcherTimer 移除。当然，从这里也可以看到，即使在业务代码里面没有对 DispatcherTimer 进行引用，但是只要这个 DispatcherTimer 还在运行，那么 DispatcherTimer 的对象就不会被释放。接着在 Dispatcher 更新计时器

```csharp
    public sealed class Dispatcher
    {
        internal void RemoveTimer(DispatcherTimer timer)
        {
            lock(_instanceLock)
            {
                if(!_hasShutdownFinished) // Could be a non-dispatcher thread, lock to read
                {
                    _timers.Remove(timer);
                    _timersVersion++;
                }
            }
            UpdateWin32Timer();
        }
    }
```

再接下来，产品大佬告诉咱，加需求。可以让开发者修改 DispatcherTimer 的计时时间，在修改 Interval 属性时，需要咱自己去更新 Dispatcher 的计时器

在 IsEnabled 开启时，如果用户修改 Interval 属性，那么需要告诉 Dispatcher 更新计时器。而如果没有开启计时器，那更新 Dispatcher 做什么

```csharp
    public class DispatcherTimer
    {
        /// <summary>
        ///     Gets or sets the time between timer ticks.
        /// </summary>
        public TimeSpan Interval
        {
            get
            {
                return _interval;
            }

            set
            {
                bool updateWin32Timer = false;

                lock(_instanceLock)
                {
                    _interval = value;

                    if(_isEnabled)
                    {
                        _dueTimeInTicks = Environment.TickCount + (int)_interval.TotalMilliseconds;
                        updateWin32Timer = true;
                    }
                }

                if(updateWin32Timer)
                {
                    _dispatcher.UpdateWin32Timer();
                }
            }
        }

        private TimeSpan _interval;
    }
```

当然了，作为对外公开的 API 还需要判断一下调皮的用户的行为。如果传入的时间是负数呢？如果传入的时间太长了，例如超过 int 的 MaxValue 也就是说这个 DispatcherTimer 是不执行的吧，是不是就需要告诉用户

```csharp
        public TimeSpan Interval
        {
            set
            {
                bool updateWin32Timer = false;
                
                if (value.TotalMilliseconds < 0)
                    throw new ArgumentOutOfRangeException("value", SR.Get(SRID.TimeSpanPeriodOutOfRange_TooSmall));

                if (value.TotalMilliseconds > Int32.MaxValue)
                    throw new ArgumentOutOfRangeException("value", SR.Get(SRID.TimeSpanPeriodOutOfRange_TooLarge));

                // 忽略代码
            }
        }
```

产品大佬还说，咱的 DispatcherTimer 是允许在后台线程启动的，毕竟不想让用户需要写 Dispatcher 调度到主线程再开启 DispatcherTimer 计时，允许在后台线程开启。如上面代码，其实咱加了很多锁了，问题也不大。这部分逻辑实现太简单了，这里就不告诉大家了

以上大概就是 DispatcherTimer 的核心逻辑，可以看到 DispatcherTimer 里面的细节还是很多的。实际的 WPF 代码里面也有很多细节部分是本文没有告诉大家的，还请大家自己去阅读 WPF 源代码

更多 DispatcherTimer 请看： [WPF 如何知道当前有多少个 DispatcherTimer 在运行](https://blog.lindexi.com/post/WPF-%E5%A6%82%E4%BD%95%E7%9F%A5%E9%81%93%E5%BD%93%E5%89%8D%E6%9C%89%E5%A4%9A%E5%B0%91%E4%B8%AA-DispatcherTimer-%E5%9C%A8%E8%BF%90%E8%A1%8C.html )

当前的 WPF 在 [https://github.com/dotnet/wpf](https://github.com/dotnet/wpf) 完全开源，使用友好的 MIT 协议，意味着允许任何人任何组织和企业任意处置，包括使用，复制，修改，合并，发表，分发，再授权，或者销售。在仓库里面包含了完全的构建逻辑，只需要本地的网络足够好（因为需要下载一堆构建工具），即可进行本地构建

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
