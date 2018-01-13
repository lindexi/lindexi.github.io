# WPF 使用RPC调用其他进程

如果在 WPF 需要用多进程通信，一个推荐的方法是 WCF ，因为 WCF 是 RPC 计算。先来讲下 RPC (Remote Procedure Call) 远程过程调用，他是通过特定协议，包括 tcp 、http 等对其他进程进行调用的技术。详细请看[百度](https://baike.baidu.com/item/%E8%BF%9C%E7%A8%8B%E8%BF%87%E7%A8%8B%E8%B0%83%E7%94%A8%E5%8D%8F%E8%AE%AE?fromtitle=RPC&fromid=609861)

<!--more-->
<!-- csdn -->

现在不会告诉大家如何使用 WCF ，下面讲的是使用 remoting 这个方法。需要知道 dotnet remoting 是已经过时的技术，建议使用 wcf 但是 wcf 部署难度比较高，对于性能要求比较高或想快速使用，建议使用 remoting 。使用方法很简单

