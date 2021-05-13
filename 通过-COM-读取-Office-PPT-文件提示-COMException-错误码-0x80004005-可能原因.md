
# 通过 COM 读取 Office PPT 文件提示 COMException 错误码 0x80004005 可能原因

使用 COM 的方式可以调用本机的 Office 组件进行 PPT 以及 Word 和 Excel 等文件的读写，在打开文件的时候，如果提示 System.Runtime.InteropServices.COMException (0x80004005) 就意味着这是一个通用的错误，没有具体的原因

<!--more-->


<!-- CreateTime:2021/5/10 19:41:43 -->

<!-- 发布 -->

调用 COM 组件，提示 `System.Runtime.InteropServices.COMException (0x80004005): Error HRESULT E_FAIL has been returned from a call to a COM component.` 表示发现通用的错误，或者未知的错误。我记录一些主要注意的事情，方便大家按照顺序去找是否此原因

## STA 线程问题

如果当前线程不是 STA 线程，那么有一些文档打开将会提示此错误

如以下代码打开 PPT 文件

```csharp
                Application = new Application();
                Presentation = Application.Presentations.Open(ppt.FullName + PASSWORD_MARK, MsoTriState.msoTrue,
                MsoTriState.msoFalse,
                MsoTriState.msoFalse);

        /// <summary>
        /// 使用密码打开ppt（如果课件无密码则正常导入，密码错误则会抛密码错误异常，这里我们使用一个密码“PASSWORD”进行解密）；详见：https://stackoverflow.com/questions/17554892/unable-to-gracefully-abort-on-unknown-password-via-microsoft-office-interop-powe
        /// </summary>
        private const string PASSWORD_MARK = "::PASSWORD::";
```

在 Presentations.Open 提示 System.Runtime.InteropServices.COMException (0x80004005) 可能是因为当前线程不是 STA 线程。判断线程方法如下

```csharp
                if (Thread.CurrentThread.GetApartmentState() == ApartmentState.STA)
                {
                    
                }
```

行为就是在 WPF 端可以调用，但是在单元测试时失败。解决方法是新建一个 STA 线程执行，新建 STA 线程方法如下

```csharp
                var thread = new Thread(() =>
                {
                    // 执行逻辑
                })
                {
                    IsBackground = true
                };
                thread.SetApartmentState(ApartmentState.STA);
```

## 后缀名不符

例如我有文件是 PPT 格式的，但是我更改了后缀名为 PPTX 格式，那么此时也将会抛出如上错误

最简单判断是 PPT 还是 PPTX 的方法就是使用压缩方法去读取，能读取的就是 PPTX 格式，否则就是 PPT 格式。当然以上方法只是简单的方法而已，对于加密的 PPTX 格式文件或者其他非 PPT 和 PPTX 格式也没有解决

更多请看 [Office 使用 OpenXML SDK 解析文档博客目录](https://blog.lindexi.com/post/Office-%E4%BD%BF%E7%94%A8-OpenXML-SDK-%E8%A7%A3%E6%9E%90%E6%96%87%E6%A1%A3%E5%8D%9A%E5%AE%A2%E7%9B%AE%E5%BD%95.html )






<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。