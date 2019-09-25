
# 如何让 USB 设备不显示安全删除硬件弹出选项

插入一个 U 盘的时候，可以在右下角找到安全删除硬件图标，点击就可以删除此硬件。如果此时插入的是一个无线网卡，也就是 USB 无线 wifi 设备，此时如果逗比点了弹出 802.11 设备那么就不能再使用无线上网了
如果我是一个硬件供应商，如何让我的设备不会显示弹出安全删除硬件弹出选项

<!--more-->


<!-- csdn -->

我不是做硬件的，对硬件懂的很少，以下是我找到的文档，如果有说错的，欢迎小伙伴告诉我

从 [Using the USB Removable Capability for Device Container Grouping](https://docs.microsoft.com/en-us/windows-hardware/drivers/install/using-the-usb-removable-capability-for-device-container-grouping )

可以找到这句话

> The USB Removable capability allows the operating system to create a device container for legacy devices.

一个 USB 设备是需要声明自己支持 Removable 的才可以在右下角使用安全删除硬件弹出选项

从 [Container IDs Generated from the Removable Device Capability](https://docs.microsoft.com/en-us/windows-hardware/drivers/install/container-ids-generated-from-the-removable-device-capability ) 可以知道，在插入即播放功能将会使用到这个功能，在设备插入的时候，通过发送 `IRP_MN_QUERY_CAPABILITIES` 获取到设备的返回信息，就可以知道这个设备支不支持移除

> The Plug and Play manager uses the removable device capability to generate a container ID for all devnodes enumerated for the physical device. The bus driver reports the removable device capability in response to an IRP_MN_QUERY_CAPABILITIES request.

那么上面说的发送信息是什么，就从[IRP_MN_QUERY_CAPABILITIES](https://docs.microsoft.com/en-us/windows-hardware/drivers/kernel/irp-mn-query-capabilities ) 可以知道，在硬件设备被枚举时，系统的 PnP 也就是插入即播放功能将会发送 `IRP_MN_QUERY_CAPABILITIES` 信息给到硬件，此时硬件收到时将会回复 `DEVICE_CAPABILITIES` 信息

> When a device is enumerated, but before the function and filter drivers are loaded for the device, the PnP manager sends an IRP_MN_QUERY_CAPABILITIES request to the parent bus driver for the device. The bus driver must set any relevant values in the DEVICE_CAPABILITIES structure and return it to the PnP manager.

硬件回复的消息请看 [DEVICE_CAPABILITIES (wdm.h)](https://docs.microsoft.com/zh-cn/windows-hardware/drivers/ddi/content/wdm/ns-wdm-_device_capabilities ) 在 PnP 询问USB设备，此时USB设备返回 DEVICE_CAPABILITIES 里面可以设置 Removable 项说明此设备支持移除。也就是想要自己的 USB 设备不能被移除，那么就声明不支持移除





<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。