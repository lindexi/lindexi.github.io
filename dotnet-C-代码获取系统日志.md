
# dotnet C# 代码获取系统日志

可以使用 EventLog 静态类读取或写入系统日志

<!--more-->


<!-- CreateTime:2021/4/14 17:48:45 -->

<!-- 发布 -->

如下面代码可以读取本机的系统日志

```csharp
        static void Main(string[] args)
        {
            foreach (var eventLog in EventLog.GetEventLogs())
            {
                foreach (EventLogEntry entry in eventLog.Entries)
                {
                    Debug.WriteLine(entry.Message);
                }
            }
        }
```

更多请看 [PowerShell 拿到最近的10个系统日志](https://blog.lindexi.com/post/PowerShell-%E6%8B%BF%E5%88%B0%E6%9C%80%E8%BF%91%E7%9A%8410%E4%B8%AA%E7%B3%BB%E7%BB%9F%E6%97%A5%E5%BF%97.html )

本文代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/8beda1f5/LejairbairwecarnelLelearnawcana) 欢迎小伙伴访问

![](http://image.acmx.xyz/lindexi%2FIMG_75762.jpg)





<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。