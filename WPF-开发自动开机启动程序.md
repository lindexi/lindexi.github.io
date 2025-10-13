
# WPF 开发自动开机启动程序

本文告诉大家如何在 WPF 开发一个可以自动启动的程序

<!--more-->


<!-- CreateTime:2018/9/2 15:10:52 -->

<!-- csdn -->

本文使用的自动开机启动方法是通过快捷方式放在启动文件夹的方式。

## 创建快捷方式

```csharp
       /// <summary>
        /// 为本程序创建一个快捷方式。
        /// </summary>
        /// <param name="lnkFilePath">快捷方式的完全限定路径。</param>
        /// <param name="args">快捷方式启动程序时需要使用的参数。</param>
        private static void CreateShortcut(string lnkFilePath, string args)
        {
            var shellType = Type.GetTypeFromProgID("WScript.Shell");
            dynamic shell = Activator.CreateInstance(shellType);
            var shortcut = shell.CreateShortcut(lnkFilePath);
            shortcut.TargetPath = Assembly.GetEntryAssembly().Location;
            shortcut.Arguments = args;
            shortcut.WorkingDirectory = AppDomain.CurrentDomain.SetupInformation.ApplicationBase;
            shortcut.Save();
        }
```

## 创建在启动文件

将快捷方式创建在启动文件夹就可以让程序开机自动启动，上面的方法已经可以传入 lnkFilePath 所以只需要设置 lnkFilePath 是启动文件夹就可以

设置的方式是 `Environment.GetFolderPath` 传入启动的文件夹

```csharp
            var startupPath = Path.Combine(
                Environment.GetFolderPath(Environment.SpecialFolder.Startup),
                "xx.lnk");
```

现在就可以让软件在开机自动启动。

参见 [使用 C# 代码创建快捷方式文件 - walterlv](https://blog.walterlv.com/post/create-shortcut-file-using-csharp.html )

## 注册表方式

除了写快捷方式到启动文件夹之外，还可以在以下注册表路径下添加启动项

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

![](https://i.loli.net/2018/09/02/5b8b8a8e69248.jpg)




<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。