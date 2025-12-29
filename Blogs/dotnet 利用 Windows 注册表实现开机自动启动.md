---
title: dotnet 利用 Windows 注册表实现开机自动启动
description: 本文记录一个开机自动启动实现方法，通过写入到注册表实现开机之后，用户登录完成之后让应用程序开机自启
tags: dotnet
category: 
---

<!-- CreateTime:2025/10/21 07:01:46 -->

<!-- 发布 -->
<!-- 博客 -->

本文将演示写入 `HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Run` 注册表路线，实现应用程序开机自动启动

核心代码如下

```csharp
static class BoostHelper
{
    /// <summary>
    /// 添加到启动项，添加到注册表，仅限 Windows 系统，写入到 HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Run 里，实现开机启动
    /// </summary>
    /// <param name="name"></param>
    /// <param name="exePath"></param>
    /// <param name="arguments"></param>
    [SupportedOSPlatform("Windows")]
    public static void AddStartup(string name, string exePath, params string[] arguments)
    {
        // 添加开机启动
        // 写入到 HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Run 里
        var runKey = Microsoft.Win32.Registry.CurrentUser.OpenSubKey("Software\\Microsoft\\Windows\\CurrentVersion\\Run", true);
        if (runKey is not null)
        {
            var valueStringBuilder = new StringBuilder();
            valueStringBuilder.Append($"\"{exePath}\"");
            foreach (var argument in arguments)
            {
                valueStringBuilder.Append($" \"{argument}\"");
            }

            string value = valueStringBuilder.ToString();

            var existingValue = runKey.GetValue(name) as string;
            if (existingValue != value)
            {
                runKey.SetValue(name, value);
                //LogMessage($"已设置开机启动，路径：{exePath}");
            }
            else
            {
                //LogMessage($"已设置过开机启动，路径：{exePath}");
            }
        }
    }

    [SupportedOSPlatform("Windows")]
    public static void RemoveStartup(string name)
    {
        var runKey = Microsoft.Win32.Registry.CurrentUser.OpenSubKey("Software\\Microsoft\\Windows\\CurrentVersion\\Run", true);
        if (runKey is not null)
        {
            runKey.DeleteValue(name, false);
            //LogMessage($"已删除开机启动，路径：{exePath}");
        }
    }
}
```

调用 BoostHelper 方法的示例代码如下

```csharp
    BoostHelper.AddStartup("Test", Environment.ProcessPath!);
```

传入的 AddStartup 的 name 参数表示一个注册表项名，合理的命名即可，没有特殊含义。同名的 name 将会相互覆盖，利用此覆盖机制即可实现更新启动路径的功能。删除时，就是直接删除对应的 name 项即可

写入之后，将在开机之后，用户登录之后，进入桌面之后启动程序。启动时使用当前用户的账户权限，且启动时机是延迟启动，相对来说比较友好，不会卡开机。这是比较常用的开机自启的注册表

在 Windows 上，还有很多其他注册表项可以用来添加开机自启，其差别的核心维度就是由什么、在什么时机启动

常见的添加开机自启的注册表项如下

- 当前用户专有的启动文件夹： `%AppData%\Microsoft\Windows\Start Menu\Programs`
- 所有用户有效的启动文件夹： `%ProgramData%\Microsoft\Windows\Start Menu\Programs`
- Userinit注册键
  - 注册表地址： `HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon`
  - 通常该注册表下面有一个`C:\Windows\system32\userinit.exe,`值，但这个键值是允许用逗号来分隔多个程序的，比如 C:\Windows\system32\userinit.exe,C:\lindexi.exe(举例)
- Explorer\Run注册键
  - 当前用户的注册表地址：`HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Policies\Explorer\Run`
  - 机器级的注册表地址：`HKEY_LOCAL_MACHINE\Software\Microsoft\Windows\CurrentVersion\Policies\Explorer\Run`
- RunServicesOnce注册键
  - 当前用户的注册表地址：`HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\RunServicesOnce`
  - 机器级的注册表地址：`HKEY_LOCAL_MACHINE\Software\Microsoft\Windows\CurrentVersion\RunServicesOnce`
  - RunServicesOnce注册键是用来启动服务的，启动时间是在用户登录之前，而且是先于其它通过注册键启动的程序
- RunServices注册键
  - 当前用户的注册表地址：`HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\RunServices`
  - 机器级的注册表地址：`HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\RunServices`
  - RunServices注册键指定的程序紧接RunServicesOnce指定的程序之后运行，启动时间也是在用户登录之前
- RunOnce注册键
  - 注册表地址：
    - `HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\RunOnce`
    - `HKEY_LOCAL_MACHINE\Software\WOW6432Node\Microsoft\Windows\CurrentVersion\RunOnce`
    - `HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\RunOnce`
  - HKEY_LOCAL_MACHINE下面的RunOnce注册键会在用户登录之后立即运行程序，运行的时机是在其它Run键指定的程序之前
  - HKEY_CURRENT_USER则会启动比较慢，它会在操作系统处理其他Run键以及“启动”文件夹的内容之后运行
- Run注册键
  - 注册表地址：
    - `HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Run`
    - `HKEY_LOCAL_MACHINE\Software\WOW6432Node\Microsoft\Windows\CurrentVersion\Run`
    - `HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Run`
  - Run是自动运行程序最常用的注册表。会先执行HKEY_LOCAL_MACHINE下的Run注册键内容，再执行HKEY_CURRENT_USER下的Run注册键内容，但两者都是在处理“启动文件夹”之前

某些流氓软件为了让任务管理器里面禁用的启动项功能失效，让自己的软件能够突破任务管理器里面的禁用启动项拦截，就会在 `计算机\HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Explorer\StartupApproved\Run` 里面删除自己的项，从而让任务管理器的禁用启动项功能失效

除了本文记录的写注册表方法之外，还可以在启动文件夹存放快捷方式实现开机自启，详细请参阅 [WPF 开发自动开机启动程序](https://blog.lindexi.com/post/WPF-%E5%BC%80%E5%8F%91%E8%87%AA%E5%8A%A8%E5%BC%80%E6%9C%BA%E5%90%AF%E5%8A%A8%E7%A8%8B%E5%BA%8F.html )

本文代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/e2a02a9aaa2148cc580cdbc91b92a5970d4b470f/Workbench/QeyeqawkayqaiWhonirikaywi) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/e2a02a9aaa2148cc580cdbc91b92a5970d4b470f/Workbench/QeyeqawkayqaiWhonirikaywi) 上，可以使用如下命令行拉取代码。我整个代码仓库比较庞大，使用以下命令行可以进行部分拉取，拉取速度比较快

先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin e2a02a9aaa2148cc580cdbc91b92a5970d4b470f
```

以上使用的是国内的 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码，将 gitee 源换成 github 源进行拉取代码。如果依然拉取不到代码，可以发邮件向我要代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin e2a02a9aaa2148cc580cdbc91b92a5970d4b470f
```

获取代码之后，进入 Workbench/QeyeqawkayqaiWhonirikaywi 文件夹，即可获取到源代码
