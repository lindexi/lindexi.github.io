# dotnet 学习 CPF 框架笔记 了解 X11 里如何获取触摸信息

本文记录我学习 CPF 框架的笔记，本文记录我阅读 CPF 框架，学习到了如何在 dotnet C# 里面获取到 X11 的触摸信息，获取到多指触摸以及触摸点的面积和触摸点压感等信息的方法

<!--more-->
<!-- CreateTime:2024/04/09 07:13:42 -->

<!-- 发布 -->
<!-- 博客 -->

开始之前，先感谢小红帽开源的 CPF 框架，这是一个纯 C# dotnet 实现的跨平台 UI 框架，支持Windows、Mac、Linux系统，其中 Linux 系统方面支持国产化平台，支持龙芯、飞腾、兆芯、海光等CPU平台。设计上和WPF一样的理念，任何控件都可以任意设计模板来实现各种效果
除了使用平台相关API之外，基本可以实现一次编写，到处运行。详细请参阅 <https://gitee.com/csharpui/CPF>

以下是用 AI 生成的 CPF 的宣传标语

> 这个CPF跨平台UI框架真是太棒了！不仅具有强大的跨平台兼容性，还拥有简洁直观的界面设计，让开发变得更加高效和便捷。无论是移动端还是桌面端，都能轻松实现一致的用户体验，实在是开发者的利器！强烈推荐给所有需要跨平台UI解决方案的开发团队！

<!-- 在开始之前需要明确一下概念，尽管创建的是 X11 的窗口，但是在 X11 时代，对触摸是没有做出定义的。需要在 X12 层才能收到触摸消息，这就是为什么本文开始说的是从 X11 X12 里获取触摸信息 -->

本文核心阅读的 CPF 代码在：<https://gitee.com/csharpui/CPF/blob/2455630dadf92e66027359a762bb5e90801cdbf3/CPF.Linux/XI2Manager.cs>

本文将从 CPF 框架里面抄出部分关键代码，在本文末尾大家可以找到本文所有的代码的下载方法

在 [学习 CPF 框架笔记 了解 X11 窗口和消息基础知识](https://blog.lindexi.com/post/%E5%AD%A6%E4%B9%A0-CPF-%E6%A1%86%E6%9E%B6%E7%AC%94%E8%AE%B0-%E4%BA%86%E8%A7%A3-X11-%E7%AA%97%E5%8F%A3%E5%92%8C%E6%B6%88%E6%81%AF%E5%9F%BA%E7%A1%80%E7%9F%A5%E8%AF%86.html ) 的基础上，假定当前已创建完成了窗口，准备好了事件监听

根据 x.org 的[官方文档](https://www.x.org/wiki/Development/Documentation/Multitouch/) 可以知道，多指触摸支持可用到 XI 2.2 的定义。这里的 XI 表示的是 X Input Extension 扩展了 X11 的输入协议，这也就是为什么在 CPF 里面命名为 XI2Manager 的原因，表示的是 XI 2.x 版本的封装逻辑

<!--
请问以上代码可以如何处理多指触摸消息

请指出以上你给出的代码里面，在哪里处理多指触摸消息。请注意，不要给出处理鼠标的代码，你必须给出的是处理多指触摸的代码，请在处理多指触摸的代码上添加注释标注

请给出 X11 处理多指触摸消息的核心代码，请注意，你要给出的是核心的代码，其他无关代码不要给出。请务必确定你给出的是处理多指触摸的代码，不要给出任何处理鼠标的代码。要求你给出的代码能够处理至少两个手指进行的触摸消息，包括如何区分输入是哪个手指

X11 的 XI 2.2 的提出日期是？
-->

开始之前，先从 CPF 或 Avalonia 里面抄足够的 P/Invoke 代码，这部分代码可以从本文末尾找到下载方法

先枚举可用设备，获取到主触摸设备，代码如下。以下代码需要开启不安全代码

```csharp
        var devices = (XIDeviceInfo*) XIQueryDevice(Display,
            (int) XiPredefinedDeviceId.XIAllMasterDevices, out int num);
        Console.WriteLine($"DeviceNumber={num}");
```

开启遍历，获取到 XIMasterPointer 设备，代码如下

```csharp
        XIDeviceInfo? pointerDevice = default;
        for (var c = 0; c < num; c++)
        {
            Console.WriteLine($"XIDeviceInfo [{c}] {devices[c].Deviceid} {devices[c].Use}");

            if (devices[c].Use == XiDeviceType.XIMasterPointer)
            {
                pointerDevice = devices[c];
                break;
            }
        }
```

如果 `pointerDevice` 不为空，则证明枚举到了主触摸输入设备。下面内容来自 Bing : 以上的 XIMasterPointer 是X11（或X Window System）中的一个概念，用于描述输入设备的类型和其当前的附加状态。当一个设备被标识为 XIMasterPointer 时，它是一个主指针。这意味着它是一个用于控制光标的输入设备，通常是鼠标。附加字段指示了与该主指针设备配对的其他设备的设备ID。具体而言：

- 如果 **use** 是 **XIMasterPointer**，那么该设备是一个**主指针**，**attachment** 指定了配对的**主键盘**的设备ID。
- 如果 **use** 是 **XIMasterKeyboard**，那么该设备是一个**主键盘**，**attachment** 指定了配对的**主指针**的设备ID。
- 如果 **use** 是 **XISlavePointer**，那么该设备是一个**从属指针**，当前连接到 **attachment** 中指定的**主指针**。
- 如果 **use** 是 **XISlaveKeyboard**，那么该设备是一个**从属键盘**，当前连接到 **attachment** 中指定的**主键盘**。
- 如果 **use** 是 **XIFloatingSlave**，那么该设备是一个**浮动从属设备**，目前未连接到任何主设备。对于浮动从属设备，**attachment** 字段的值是未定义的。

拿到主指针设备之后，向其注册触摸事件订阅，代码如下

```csharp
            var multiTouchEventTypes = new List<XiEventType>
            {
                XiEventType.XI_TouchBegin,
                XiEventType.XI_TouchUpdate,
                XiEventType.XI_TouchEnd
            };

            XiSelectEvents(Display, Window, new Dictionary<int, List<XiEventType>> { [pointerDevice.Value.Deviceid] = multiTouchEventTypes });
```

以上的 XiSelectEvents 定义如下

```csharp
        [DllImport(libXInput)]
        public static extern Status XISelectEvents(
            IntPtr dpy,
            IntPtr win,
            XIEventMask* masks,
            int num_masks
        );

        public static Status XiSelectEvents(IntPtr display, IntPtr window, Dictionary<int, List<XiEventType>> devices)
        {
            var masks = stackalloc int[devices.Count];
            var emasks = stackalloc XIEventMask[devices.Count];
            int c = 0;
            foreach (var d in devices)
            {
                foreach (var ev in d.Value)
                    XISetMask(ref masks[c], ev);
                emasks[c] = new XIEventMask
                {
                    Mask = &masks[c],
                    Deviceid = d.Key,
                    MaskLen = XiEventMaskLen
                };
                c++;
            }


            return XISelectEvents(display, window, emasks, devices.Count);
        }
```

如此即可在 XNextEvent 里面收到触摸消息

```csharp
            var xNextEvent = XNextEvent(Display, out XEvent @event);
```

但是触摸事件是不能直接通过 `@event` 的 type 进行判断的，如下面代码是不能用于判断接收到了触摸消息的

```csharp
            int type = (int) @event.type;

            if (type is (int) XiEventType.XI_TouchBegin
                    or (int) XiEventType.XI_TouchUpdate
                    or (int) XiEventType.XI_TouchEnd)
            {
                Console.WriteLine($"Touch {(XiEventType) type} {@event.MotionEvent.x} {@event.MotionEvent.y}");
            }
```

以上代码的控制台输出将不会执行。正确的获取触摸事件消息，需要从 `@event` 的 GenericEventCookie 数据里面获取。即先判断输入的类型是否 GenericEvent 类型，再获取其 GenericEventCookie 的 data 数据部分，进一步判断 data 的 `evtype` 是否 XI_Touch 系列即可，代码如下

```csharp
            if (@event.type == XEventName.GenericEvent)
            {
                void* data = &@event.GenericEventCookie;
                /*
                 bing:
                `XGetEventData` 是一个用于 **X Window System** 的函数，其主要目的是通过 **cookie** 来检索和释放附加的事件数据。让我们来详细了解一下：

                   - **函数名称**：`XGetEventData`
                   - **功能**：检索通过 **cookie** 存储的附加事件数据。
                   - **参数**：
                       - `display`：指定与 X 服务器的连接。
                       - `cookie`：指定要释放或检索数据的 **cookie**。
                   - **结构体**：`XGenericEventCookie`
                       - `type`：事件类型。
                       - `serial`：事件序列号。
                       - `send_event`：是否为发送事件。
                       - `display`：指向 X 服务器的指针。
                       - `extension`：扩展信息。
                       - `evtype`：事件类型。
                       - `cookie`：唯一标识此事件的 **cookie**。
                       - `data`：事件数据的指针，在调用 `XGetEventData` 之前未定义。
                   - **描述**：某些扩展的 `XGenericEvents` 需要额外的内存来存储信息。对于这些事件，库会返回一个具有唯一标识此事件的 **cookie** 的 `XGenericEventCookie`。直到调用 `XGetEventData`，`XGenericEventCookie` 的数据指针是未定义的。`XGetEventData` 函数检索给定 **cookie** 的附加数据。不需要与服务器进行往返通信。如果 **cookie** 无效或事件不是由 **cookie** 处理程序处理的事件，则返回 `False`。如果 `XGetEventData` 返回 `True`，则 **cookie** 的数据指针指向包含事件信息的内存。客户端必须调用 `XFreeEventData` 来释放此内存。对于同一事件 **cookie** 的多次调用，`XGetEventData` 返回 `False`。`XFreeEventData` 函数释放与 **cookie** 关联的数据。客户端必须对使用 `XGetEventData` 获得的每个 **cookie** 调用 `XFreeEventData`。
                   - **注意事项**：
                       - 如果 **cookie** 已通过 `XNextEvent` 返回给客户端，但其数据尚未通过 `XGetEventData` 检索，则该 **cookie** 被定义为未声明。后续对 `XNextEvent` 的调用可能会释放与未声明 **cookie** 关联的内存。
                       - 多线程的 X 客户端必须确保在下一次调用 `XNextEvent` 之前调用 `XGetEventData`。

                   更多信息，请参阅 [XGetEventData 文档](https://www.x.org/releases/X11R7.6/doc/man/man3/XGetEventData.3.xhtml)。¹²

                   源: 与必应的对话， 2024/4/7
                   (1) XGetEventData - X Window System. https://www.x.org/releases/X11R7.6/doc/man/man3/XGetEventData.3.xhtml.
                   (2) XGetEventData(3) — libX11-devel. https://man.docs.euro-linux.com/EL%209/libX11-devel/XGetEventData.3.en.html.
                   (3) X11R7.7 Manual Pages: Section 3: Library Functions - X Window System. https://www.x.org/releases/X11R7.7/doc/man/man3/.
                 */
                XGetEventData(Display, data);
                try
                {
                    var xiEvent = (XIEvent*) @event.GenericEventCookie.data;
                    if (xiEvent->evtype == XiEventType.XI_DeviceChanged)
                    {
                    }

                    if (xiEvent->evtype is
                        XiEventType.XI_ButtonRelease
                        or XiEventType.XI_ButtonRelease
                        or XiEventType.XI_Motion
                        or XiEventType.XI_TouchBegin
                        or XiEventType.XI_TouchUpdate
                        or XiEventType.XI_TouchEnd)
                    {
                        var xiDeviceEvent = (XIDeviceEvent*) xiEvent;

                        var timestamp = (ulong) xiDeviceEvent->time.ToInt64();
                        var state = (XModifierMask) xiDeviceEvent->mods.Effective;

                        // 对应 WPF 的 TouchId 是 xiDeviceEvent->detail 字段
                        Console.WriteLine($"[{xiEvent->evtype}][{xiDeviceEvent->deviceid}][{xiDeviceEvent->sourceid}] detail={xiDeviceEvent->detail} timestamp={timestamp} {state} X={xiDeviceEvent->event_x} Y={xiDeviceEvent->event_y} root_x={xiDeviceEvent->root_x} root_y={xiDeviceEvent->root_y}");
                    }
                }
                finally
                {
                    /*
                     bing:
                       如果不调用 `XFreeEventData`，会导致一些潜在问题和资源泄漏。让我详细解释一下：

                       - **资源泄漏**：`XGetEventData` 函数会分配内存来存储事件数据。如果不调用 `XFreeEventData` 来释放这些内存，会导致内存泄漏。这可能会在长时间运行的应用程序中累积，最终导致内存耗尽或应用程序崩溃。

                       - **未定义行为**：如果不调用 `XFreeEventData`，则 `XGenericEventCookie` 的数据指针将保持未定义状态。这意味着您无法访问事件数据，从而可能导致应用程序中的错误或不一致性。

                       - **性能问题**：如果不释放事件数据，系统可能会在内部维护大量未释放的内存块，从而影响性能。

                       因此，为了避免这些问题，务必在使用 `XGetEventData` 获取事件数据后调用 `XFreeEventData` 来释放内存。这是良好的编程实践，有助于确保应用程序的稳定性和性能。
                     */
                    XFreeEventData(Display, data);
                }
```

如此即可获取到触摸的 X 和 Y 点坐标，以及通过 detail 区分多指触摸。这里的 detail 就是对应 WPF 的 TouchId 之类的属性。以上的 `event_x` 和 `event_y` 指的是窗口坐标系的，相对于当前窗口的左上角，而 `root_x` 和 `root_y` 是屏幕坐标系的，由于我这里没有多个屏幕，没有测试多屏幕的行为

以上的触摸消息里面，在 XIDeviceEvent 的 valuators 里面可能带着额外的触摸数据，比如触摸的面积和触摸的压感值。这里需要额外说明的是触摸面积这里我指的是对应 WPF 这边的触摸的宽度和高度信息，但是在 X 系列里面，是采用椭圆面积方式，通过 `Touch Major` 和 `Touch Minor` 分别定义椭圆的长轴和短轴。即 ABS_MT_TOUCH_MAJOR 和 ABS_MT_TOUCH_MINOR 的定义。这个定义看起来和安卓手机上的定义有些类似，详细请参阅[安卓触摸设备文档](https://source.android.google.cn/docs/core/interaction/input/touch-devices)

为了获取 valuators 里面包含的触摸面积信息以及触摸压感信息，需要提前通过 XInternAtom 获取当前 XInput 对于触摸额外数据的定义，或者准确说是 Atom 原子标识符，代码如下

```csharp
        var touchMajorAtom = XInternAtom(Display, "Abs MT Touch Major", false);
        var touchMinorAtom = XInternAtom(Display, "Abs MT Touch Minor", false);
        var pressureAtom = XInternAtom(Display, "Abs MT Pressure", false);
```

传入给到 XInternAtom 的字符串是大小写敏感的，可不要传错哦。可以通过在测试的设备上输入 xinput 命令，查看当前的设备的原子对应，以及将以上代码的 `touchMajorAtom` 等参数打印出来，查看是否相同，如相同则证明代码编写正确

```csharp
        Console.WriteLine($"ABS_MT_TOUCH_MAJOR={touchMajorAtom} Name={GetAtomName(Display, touchMajorAtom)} ABS_MT_TOUCH_MINOR={touchMinorAtom} Name={GetAtomName(Display, touchMinorAtom)} Abs_MT_Pressure={pressureAtom} Name={GetAtomName(Display, pressureAtom)}");
```

对应在控制台输入 xinput 可以看到大概如下的输出内容。括号里面的数字就期望能够与上面代码控制台输出的 Atom 值相同。如 `ABS_MT_TOUCH_MAJOR={touchMajorAtom}` 这里的 `touchMajorAtom` 就应该预期与下面控制台输出的 `"Abs MT Touch Major" (277)` 的 277 相同

```
> xinput
...
	Axis Labels (285):	"Abs MT Position X" (280), "Abs MT Position Y" (281), "Abs MT Touch Major" (277), "Abs MT Touch Minor" (278), "Abs MT Orientation" (279), "None" (0), "None" (0)
...	
```

由于不同的触摸设备在描述符信息上可能添加了不同的功能支持程度，有些触摸设备，如我拿到的一个 DELL 的触摸屏，就不支持触摸的宽度和高度信息。这些可以通过读取上文获取到的指针设备 `pointerDevice` 局部变量的 Classes 字段，从而了解当前的设备支持哪些功能

```csharp
            var valuators = new List<XIValuatorClassInfo>();
            var scrollers = new List<XIScrollClassInfo>();

            for (int i = 0; i < pointerDevice.Value.NumClasses; i++)
            {
                var xiAnyClassInfo = pointerDevice.Value.Classes[i];
                if (xiAnyClassInfo->Type == XiDeviceClass.XIValuatorClass)
                {
                    valuators.Add(*((XIValuatorClassInfo**) pointerDevice.Value.Classes)[i]);
                }
                else if (xiAnyClassInfo->Type == XiDeviceClass.XIScrollClass)
                {
                    scrollers.Add(*((XIScrollClassInfo**) pointerDevice.Value.Classes)[i]);
                }
            }
```

完成以上代码之后，可以尝试输出一下，输出当前设备支持的输入信息

```csharp
            foreach (XIValuatorClassInfo xiValuatorClassInfo in valuators)
            {
                var label = xiValuatorClassInfo.Label;
                // 不能通过 Marshal.PtrToStringAnsi 读取 Label 的值 读取不到
                //Marshal.PtrToStringAnsi(xiValuatorClassInfo.Label);
                Console.WriteLine($"[Valuator] [{GetAtomName(Display, label)}] Label={label} Type={xiValuatorClassInfo.Type} Sourceid={xiValuatorClassInfo.Sourceid} Number={xiValuatorClassInfo.Number} Min={xiValuatorClassInfo.Min} Max={xiValuatorClassInfo.Max} Value={xiValuatorClassInfo.Value} Resolution={xiValuatorClassInfo.Resolution} Mode={xiValuatorClassInfo.Mode}");
            }
```

以上代码的 GetAtomName 的定义如下

```csharp
        [DllImport(libX11)]
        public static extern IntPtr XGetAtomName(IntPtr display, IntPtr atom);

        public static string? GetAtomName(IntPtr display, IntPtr atom)
        {
            var ptr = XGetAtomName(display, atom);
            if (ptr == IntPtr.Zero)
                return null;
            var s = Marshal.PtrToStringAnsi(ptr);
            XFree(ptr);
            return s;
        }
```

拿到 `List<XIValuatorClassInfo>` 之后，即可在后续收到触摸消息时，用 XIValuatorClassInfo 的 Number 字段与触摸的 valuators 的 Mask 对比，从而拿到当前的触摸额外信息

具体的获取触摸额外信息的方法如下，先创建触摸额外信息的 valuator 字典。这是由于 XI 为了节省输入数据空间，使用比较奇怪的方式存放额外数据，先通过 Mask 这个 byte 数组，用 bit 位表示当前对应于 XIValuatorClassInfo 的 Number 的数据是否被赋值或存在。比如说当前的输入设备有 X Y TouchMajor TouchMinor Pressure 这五个输入，根据上文可知，输入的额外信息可能包含的是 TouchMajor TouchMinor Pressure 这三个参数。在某次输入数据里面，只有 Pressure 参数有值，那此时的输入数据内容大概会是如此：

- 先是 Mask 数组只有一项，一个 byte 即可表示 8 个 bit 了
- 假定 `pressureAtom` 的 Number 刚好是 2 的值，即 TouchMajor 是 0 的值，而 TouchMinor 是 1 的值
- 那么 Mask 数组里面的唯一一个 byte 数据就是 0010_0000 的掩码值
- 对应的 Values 数组则也只存放一个 double 元素，表示的就是 Pressure 压感值

根据以上的例子数据，可以看到咱需要将 valuators 解开的最简方式就是存放字典，即通过 Mask 关联到 XIValuatorClassInfo 的 Number 字段，作为 Key 值。将 Values 放入到对应的槽内。当然了，不使用字典，使用一个数组也是可以的，只是数组的内容可能比较稀疏，可能实际大部分空间都是浪费的

以下是创建 valuator 字典的代码

```csharp
                        var valuatorDictionary = new Dictionary<int, double>();
                        var values = xiDeviceEvent->valuators.Values;
                        for (var c = 0; c < xiDeviceEvent->valuators.MaskLen * 8/*一个 Byte 有 8 个 bit，以下 XIMaskIsSet 是按照 bit 进行判断的*/; c++)
                        {
                            if (XIMaskIsSet(xiDeviceEvent->valuators.Mask, c))
                            {
                            	// 只有 Mask 存在值的，才能获取 Values 的值
                                valuatorDictionary[c] = *values;
                                values++;
                            }
                        }
```

可以通过以下的测试代码了解当前的触摸输入额外数据分别有哪些

```csharp
                        foreach (var (key, value) in valuatorDictionary)
                        {
                            var xiValuatorClassInfo = valuators.FirstOrDefault(t => t.Number == key);

                            var label = GetAtomName(Display, xiValuatorClassInfo.Label);

                            if (xiValuatorClassInfo.Label == touchMajorAtom)
                            {
                                label = "TouchMajor";
                            }
                            else if (xiValuatorClassInfo.Label == touchMinorAtom)
                            {
                                label = "TouchMinor";
                            }
                            else if (xiValuatorClassInfo.Label == pressureAtom)
                            {
                                label = "Pressure";
                            }

                            Console.WriteLine($"[Valuator] [{label}] Label={xiValuatorClassInfo.Label} Type={xiValuatorClassInfo.Type} Sourceid={xiValuatorClassInfo.Sourceid} Number={xiValuatorClassInfo.Number} Min={xiValuatorClassInfo.Min} Max={xiValuatorClassInfo.Max} Value={xiValuatorClassInfo.Value} Resolution={xiValuatorClassInfo.Resolution} Mode={xiValuatorClassInfo.Mode} Value={value}");
                        }
```

通过 XIValuatorClassInfo 的 Number 字段与 Key 判断，即可了解当前的触摸额外数据对应的是哪个维度的参数。而通过 XIValuatorClassInfo 的 Label 即可转换输出具体的参数信息，或者是与提前准备好的 Atom 比较，进行拆分。如以上代码就与提前准备好的 `touchMajorAtom` 等变量进行对比，从而拆分出具体的参数

通过以上代码即可获取到触摸的信息，包括用来触摸的面积和触摸的压感等信息

本文代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/43711cd55b54616e0d75a70d61dec5591151ad2b/BujeeberehemnaNurgacolarje) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/43711cd55b54616e0d75a70d61dec5591151ad2b/BujeeberehemnaNurgacolarje) 上，可以使用如下命令行拉取代码

先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin 43711cd55b54616e0d75a70d61dec5591151ad2b
```

以上使用的是 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码，将 gitee 源换成 github 源进行拉取代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin 43711cd55b54616e0d75a70d61dec5591151ad2b
```

获取代码之后，进入 BujeeberehemnaNurgacolarje 文件夹，即可获取到源代码

参考文档：

- <https://www.x.org/wiki/Development/Documentation/Multitouch/>
- <https://en.wikipedia.org/wiki/X_Window_System>
- <https://www.x.org/releases/X11R7.6/doc/man/man3/XIQueryDevice.3.xhtml>
- <https://www.x.org/releases/X11R7.6/doc/man/man3/XGetEventData.3.xhtml>
- <https://www.kernel.org/doc/html/latest/input/multi-touch-protocol.html>
- <https://source.android.google.cn/docs/core/interaction/input/touch-devices>

对应的，我修复了 Avalonia 的触摸问题，详细请参阅 <https://github.com/AvaloniaUI/Avalonia/pull/15297> <https://github.com/AvaloniaUI/Avalonia/pull/15283>