# win10 uwp 手动锁Bitlocker


## bitlocker

Windows BitLocker驱动器加密通过加密Windows操作系统卷上存储的所有数据可以更好地保护计算机中的数据。BitLocker使用TPM帮助保护Windows操作系统和用户数据，并帮助确保计算机即使在无人参与、丢失或被盗的情况下也不会被篡改。 BitLocker还可以在没有TPM的情况下使用。
<!--more-->
<!-- CreateTime:2018/2/13 17:23:03 -->


<div id="toc"></div>

win10有新的方法，我们可以使用新的。

我们可以在计算机管理，压缩磁盘，分区一个几百M的分区，然后使用Bitlocker锁。

但是在我们解锁后，直到重启才可以锁上。

那么在我们离开的时候如何锁上。

## 手动锁

自动锁Bitlocker是不能的，但是我们可以有简单方法去锁

建立`*.bat`

```csharp
manage-bde.exe 盘符: -lock
```
管理员运行

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。


