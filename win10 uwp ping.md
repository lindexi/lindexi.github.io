# win10 uwp ping

有时需要进行 ping ，但是ms没有给一个类可以进行 ping

本文：如何使用 ping

<!--more-->
<!-- CreateTime:2018/8/10 19:17:19 -->


<!-- csdn -->

不管什么，大概没有人还不知道什么是 ping，如果不知道的话，请百度一下，虽然百度很垃圾，不过找这个还是很简单。

简单的方法是使用大神写的

打开 Nuget ，搜索 System.Net.Ping ，接下来就是很简单

第二个方法是使用下面代码，注意把他放在一个函数，这里测试的是 lindexi.oschina.io


```csharp
               HostName host = new HostName("lindexi.oschina.io");
            var eps = await DatagramSocket.GetEndpointPairsAsync(host , "80");
            if(eps.Count >= 1)
            {
                return true;
            }
            else
            {
                return false;
            }
```


参见：http://stackoverflow.com/questions/37300532/ping-class-not-available-in-uwp

![](http://image.acmx.xyz/34fdad35-5dfe-a75b-2b4b-8c5e313038e2%2F201792392937.jpg)

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。 