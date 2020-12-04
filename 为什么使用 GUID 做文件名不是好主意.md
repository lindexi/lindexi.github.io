# 为什么使用 GUID 做文件名不是好主意

在创建随机文件使用的时候，文件的命名是神坑，我看到一些代码里面使用 GUID 作为文件名，这不是一个好主意。推荐的做法应该使用 Path.GetRandomFileName 方法

<!--more-->
<!-- CreateTime:7/1/2020 3:33:09 PM -->



为什么使用 Guid 作为文件名不是一个好主意，有以下原因

## 文件名冲突

有小伙伴认为使用 Guid 作为文件名就一定不会存在冲突，不好意思，如果你是工程师，那么应该会遇到一些非酋的用户，这部分用户将会遇到使用 Guid 创建的文件也重复的问题

有小伙伴会说，创建 Guid 不是会根据网卡还有时间等超级多的内容创建的？其实涉及的越多，就越不可靠

因此在需要创建新文件的时候，请不要认为使用 Guid 创建的就是不会冲突的文件，一定是新文件

## 文件长度

默认在 Windows 下的文件名加上路径的长度最长是 260 个字符，而一个 Guid 一般长度是 36 个字符，因此如果来个 7 层文件夹就炸了。有趣的是 AppData 文件夹的路径大部分用户差不多有 30 个字符以上，如果再拼接上应用名等，此时差不多也有 36 个字符

使用 Guid 作为文件名的小伙伴，小心也有小伙伴用来做文件夹名

那么使用 Path.GetRandomFileName 有什么优势

## 文件名更短

其实 Path.GetRandomFileName 仅仅是返回随机的字符串，这个字符串你用来做文件名也好，用来做文件夹名也好。官方的推荐是这个字符串仅仅只用来做文件或文件夹名，不要用来做其他安全相关的用途

默认的 Path.GetRandomFileName 返回的字符串长度是 12 个字符，比 Guid 小3倍，也就是差不多到 21 层的文件夹才会炸

## 更高的性能

创建 Path.GetRandomFileName 的速度比 Guid 的速度快，原因是 Path.GetRandomFileName 需要的计算量很小。就等小伙伴有空帮我测试一下啦

## 安全性不变

理论上 Path.GetRandomFileName 生成的文件冲突和使用 Guid 的文件冲突是几乎等价的，或者说在一个数量级，尽管 Guid 的文件名更长

因此使用 Path.GetRandomFileName 代替 Guid 创建随机文件看起来特别有优势。但依然使用 GetRandomFileName 有一个不足，或者说他的一个功能反而不是咱需要的。这就是 GetRandomFileName 创建的文件会加上诡异的后缀，如 tochgevm.edn
 这个文件

如果有一些垃圾代码只是判断存在一个 `.` 作为文件后缀名的，和这些代码对接也许就会炸

推荐的创建文件名的方法如下

```csharp
        private static string GetNewFileName(string folder)
        {
            while (true)
            {
                var file = Path.GetRandomFileName();
                file = Path.Combine(folder, file);
                if (!File.Exists(file))
                {
                    return file;
                }
            }
        }
```

上面代码依然无法解决多线程写文件的文件安全问题，依然可以在多进程或多线程获取文件名时文件不存在，但是在准备创建文件时，发现文件存在了

如果你发现使用上面代码遇到文件存在的坑时，请联系我，我要请你买彩票

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
