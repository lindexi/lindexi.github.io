
# dotnet 麒麟 Kylin 的 X11 应用开发记录

本文记录我在麒麟 Kylin 的 X11 应用开发过程的经验

<!--more-->


<!-- CreateTime:2024/11/26 08:40:32 -->

<!-- 发布 -->
<!-- 博客 -->

## 系统版本信息

本文面向的麒麟系统版本信息如下：

```bash
$ cat /etc/os-release
NAME="Kylin"
VERSION="银河麒麟桌面操作系统（教育版）V10"
VERSION_US="Kylin Linux Desktop EDU V10"
ID=kylin
ID_LIKE=debian
PRETTY_NAME="Kylin V10 SP1"
VERSION_ID="v10"
HOME_URL="http://www.kylinos.cn/"
SUPPORT_URL="http://www.kylinos.cn/support/technology.html"
BUG_REPORT_URL="http://www.kylinos.cn/"
PRIVACY_POLICY_URL="http://www.kylinos.cn"
VERSION_CODENAME=kylin
UBUNTU_CODENAME=kylin
PROJECT_CODENAME=V10SP1-General-Edu
KYLIN_RELEASE_ID="2403"
```

系统为从麒麟拿到的系统，没有经过什么更改。如无特殊说明，本文均采用此系统版本

## 读取麒麟系统的各项版本信息

[读取麒麟系统的各项版本信息](https://blog.lindexi.com/post/%E8%AF%BB%E5%8F%96%E9%BA%92%E9%BA%9F%E7%B3%BB%E7%BB%9F%E7%9A%84%E5%90%84%E9%A1%B9%E7%89%88%E6%9C%AC%E4%BF%A1%E6%81%AF.html )
<!-- [读取麒麟系统的各项版本信息 - lindexi - 博客园](https://www.cnblogs.com/lindexi/p/18527091 ) -->

## 图标在任务栏上

不能在 MapWindow 之后，通过发送 ClientMessage 消息的方式，通过 `_NET_WM_STATE_SKIP_TASKBAR` 设置应用不在任务栏显示图标

必须要在 MapWindow 之前完成 `_NET_WM_STATE_SKIP_TASKBAR` 的设置。由于在窗口 Map 之前调用，不能利用桌面管理器辅助设置，需要通过 XChangeProperty 进行设置

在麒麟系统让图标不在任务栏上显示的方法如下

```csharp
        var _NET_WM_STATE_SKIP_TASKBAR = XInternAtom(Display, "_NET_WM_STATE_SKIP_TASKBAR", false);
        ChangeWMAtomsByXChangeProperty(true, _NET_WM_STATE_SKIP_TASKBAR);

        XMapWindow(Display, Window);
        XFlush(Display);

    private unsafe void ChangeWMAtomsByXChangeProperty(bool enable, params IntPtr[] atoms)
    {
        var wmState = XInternAtom(Display, "_NET_WM_STATE", true);
        XGetWindowProperty(Display, Window, wmState, IntPtr.Zero, new IntPtr(256),
            false, (IntPtr) Atom.XA_ATOM, out _, out _, out var nitems, out _,
            out var prop);

        var ptr = (IntPtr*) prop.ToPointer();
        var newAtoms = new HashSet<IntPtr>();
        for (var c = 0; c < nitems.ToInt64(); c++)
        {
            newAtoms.Add(*ptr);
            ptr++;
        }

        XFree(prop);
        foreach (var atom in atoms)
        {
            if (enable)
            {
                newAtoms.Add(atom);
            }
            else
            {
                newAtoms.Remove(atom);
            }
        }

        XChangeProperty(Display, Window, wmState, (IntPtr) Atom.XA_ATOM, 32,
            PropertyMode.Replace, newAtoms.ToArray(), newAtoms.Count);
    }
```

以上的 ChangeWMAtomsByXChangeProperty 方法的实现是从 Avalonia 里面抄的，且是经过 walterlv 在 <https://github.com/AvaloniaUI/Avalonia/pull/16110> 修复之后的代码

## 拦截问题

[记 Kylin 麒麟系统安全中心拦截导致 dotnet sdk 找不到 OpenSsl 构建失败](https://blog.lindexi.com/post/%E8%AE%B0-Kylin-%E9%BA%92%E9%BA%9F%E7%B3%BB%E7%BB%9F%E5%AE%89%E5%85%A8%E4%B8%AD%E5%BF%83%E6%8B%A6%E6%88%AA%E5%AF%BC%E8%87%B4-dotnet-sdk-%E6%89%BE%E4%B8%8D%E5%88%B0-OpenSsl-%E6%9E%84%E5%BB%BA%E5%A4%B1%E8%B4%A5.html )
<!-- [记 Kylin 麒麟系统安全中心拦截导致 dotnet sdk 找不到 OpenSsl 构建失败 - lindexi - 博客园](https://www.cnblogs.com/lindexi/p/18514833 ) -->

[dotnet 记龙芯麒麟教育版安全中心拦截文件 导致 docker 内 CI CD 构建失败](https://blog.lindexi.com/post/dotnet-%E8%AE%B0%E9%BE%99%E8%8A%AF%E9%BA%92%E9%BA%9F%E6%95%99%E8%82%B2%E7%89%88%E5%AE%89%E5%85%A8%E4%B8%AD%E5%BF%83%E6%8B%A6%E6%88%AA%E6%96%87%E4%BB%B6-%E5%AF%BC%E8%87%B4-docker-%E5%86%85-CI-CD-%E6%9E%84%E5%BB%BA%E5%A4%B1%E8%B4%A5.html )
<!-- [dotnet 记龙芯麒麟教育版安全中心拦截文件 导致 docker 内 CI CD 构建失败 - lindexi - 博客园](https://www.cnblogs.com/lindexi/p/18545167 ) -->

## 更多相关博客

- [dotnet 8 版本与银河麒麟V10和UOS系统的 glibc 兼容性](https://blog.lindexi.com/post/dotnet-8-%E7%89%88%E6%9C%AC%E4%B8%8E%E9%93%B6%E6%B2%B3%E9%BA%92%E9%BA%9FV10%E5%92%8CUOS%E7%B3%BB%E7%BB%9F%E7%9A%84-glibc-%E5%85%BC%E5%AE%B9%E6%80%A7.html ) <!-- [dotnet 8 版本与银河麒麟V10和UOS系统的 glibc 兼容性 - lindexi - 博客园](https://www.cnblogs.com/lindexi/p/18161216 ) -->

更多国产化相关开发博客，请参阅 [博客导航](https://blog.lindexi.com/post/%E5%8D%9A%E5%AE%A2%E5%AF%BC%E8%88%AA.html )




<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。