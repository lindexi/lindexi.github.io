
# dotnet 执行 docker 容器 error MSB4018 CreateAppHost 任务意外失败可能原因

在使用 ASP.NET Core 的 docker 调试的时候，在生成的这一步提示 C:\Program Files\dotnet\sdk\3.1.201\Sdks\Microsoft.NET.Sdk\targets\Microsoft.NET.Sdk.targets(424,5): error MSB4018: “CreateAppHost”任务意外失败 可能的原因是 docker 内之前的容器没有关闭

<!--more-->


<!-- CreateTime:2020/8/28 8:44:52 -->

<!-- 发布 -->

关注输出窗口，如果可以看到如下内容，那么就是 docker 内存在上次运行的容器没有关闭

```
1>docker exec -i 93b62c811acccda3232d8a18072f54991fc03198f646b810f8da08351d46daf5 /bin/sh -c "if PID=$(pidof dotnet); then kill $PID; fi"
1>/bin/sh: pidof: command not found
1>C:\Users\Lindexi\.nuget\packages\microsoft.visualstudio.azure.containers.tools.targets\1.10.8\build\Container.targets(138,5): warning CTP1006: 未能在容器中停止应用程序。由于文件正在使用中，因此生成可能会失败。
1>C:\Users\Lindexi\.nuget\packages\microsoft.visualstudio.azure.containers.tools.targets\1.10.8\build\Container.targets(138,5): warning CTP1006: Docker 命令失败，退出代码为 0。
1>C:\Users\Lindexi\.nuget\packages\microsoft.visualstudio.azure.containers.tools.targets\1.10.8\build\Container.targets(138,5): warning CTP1006: /bin/sh: pidof: command not found
```

此时 VS 的错误提示内容如下

```
11>C:\Program Files\dotnet\sdk\3.1.201\Sdks\Microsoft.NET.Sdk\targets\Microsoft.NET.Sdk.targets(424,5): error MSB4018: “CreateAppHost”任务意外失败。
11>C:\Program Files\dotnet\sdk\3.1.201\Sdks\Microsoft.NET.Sdk\targets\Microsoft.NET.Sdk.targets(424,5): error MSB4018: Microsoft.NET.HostModel.HResultException: 8007006E
11>C:\Program Files\dotnet\sdk\3.1.201\Sdks\Microsoft.NET.Sdk\targets\Microsoft.NET.Sdk.targets(424,5): error MSB4018:    在 Microsoft.NET.HostModel.ResourceUpdater.Update()
11>C:\Program Files\dotnet\sdk\3.1.201\Sdks\Microsoft.NET.Sdk\targets\Microsoft.NET.Sdk.targets(424,5): error MSB4018:    在 Microsoft.NET.HostModel.AppHost.HostWriter.<>c__DisplayClass2_0.<CreateAppHost>g__UpdateResources|1()
11>C:\Program Files\dotnet\sdk\3.1.201\Sdks\Microsoft.NET.Sdk\targets\Microsoft.NET.Sdk.targets(424,5): error MSB4018:    在 Microsoft.NET.HostModel.RetryUtil.RetryOnWin32Error(Action func)
11>C:\Program Files\dotnet\sdk\3.1.201\Sdks\Microsoft.NET.Sdk\targets\Microsoft.NET.Sdk.targets(424,5): error MSB4018:    在 Microsoft.NET.HostModel.AppHost.HostWriter.CreateAppHost(String appHostSourceFilePath, String appHostDestinationFilePath, String appBinaryFilePath, Boolean windowsGraphicalUserInterface, String assemblyToCopyResorcesFrom)
11>C:\Program Files\dotnet\sdk\3.1.201\Sdks\Microsoft.NET.Sdk\targets\Microsoft.NET.Sdk.targets(424,5): error MSB4018:    在 Microsoft.NET.Build.Tasks.CreateAppHost.ExecuteCore()
11>C:\Program Files\dotnet\sdk\3.1.201\Sdks\Microsoft.NET.Sdk\targets\Microsoft.NET.Sdk.targets(424,5): error MSB4018:    在 Microsoft.NET.Build.Tasks.TaskBase.Execute()
11>C:\Program Files\dotnet\sdk\3.1.201\Sdks\Microsoft.NET.Sdk\targets\Microsoft.NET.Sdk.targets(424,5): error MSB4018:    在 Microsoft.Build.BackEnd.TaskExecutionHost.Microsoft.Build.BackEnd.ITaskExecutionHost.Execute()
11>C:\Program Files\dotnet\sdk\3.1.201\Sdks\Microsoft.NET.Sdk\targets\Microsoft.NET.Sdk.targets(424,5): error MSB4018:    在 Microsoft.Build.BackEnd.TaskBuilder.<ExecuteInstantiatedTask>d__26.MoveNext()
```

解决方法就是通过命令结束 docker 容器

使用 `docker ps` 命令找到所有在运行的实例，使用 `docker kill` 命令杀掉正在运行的实例，重新在 VS 按下 F5 开始调试就可以

关于 docker 命令请看 [docker常用命令-docker kill_OneZeroTwoFour-CSDN博客](https://blog.csdn.net/zhangzehai2234/article/details/102810772 )

[VisualStudio 解决首次调试 docker 的 vs2017u5 exists, deleting 太慢问题](https://blog.lindexi.com/post/VisualStudio-%E8%A7%A3%E5%86%B3%E9%A6%96%E6%AC%A1%E8%B0%83%E8%AF%95-docker-%E7%9A%84-vs2017u5-exists,-deleting-%E5%A4%AA%E6%85%A2%E9%97%AE%E9%A2%98.html)





<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。