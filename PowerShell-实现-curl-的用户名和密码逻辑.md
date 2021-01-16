
# PowerShell 实现 curl 的用户名和密码逻辑

在使用 curl 时，可以采用 -u 加上用户名和密码，这个对应在 PowerShell 也就是不到 10 句话的事情

<!--more-->


<!-- 发布 -->

假定使用 curl 输入的是如下代码

```
curl -ulindexi:AP7doYUzM7WApXobRb7X9qgURCF -T "E:\lindexi\doubi.exe" "https://blog.lindexi.com/artifactory/doubi.exe"
```

通过上面代码可以给我的存储服务上传文件

此时的 `-ulindexi:AP7doYUzM7WApXobRb7X9qgURCF` 的含义就是 `-u <用户名:密码>` 在对应的 Http 里面就是在 Head 的 Authorization 加入信息

在 PowerShell 中，按照规范需要传入一段 base64 的字符，于是代码如下

```
$encodedAuthString =  [System.Convert]::ToBase64String([System.Text.Encoding]::ASCII.GetBytes("lindexi:AP7doYUzM7WApXobRb7X9qgURCF")) 
$Headers = 
@{
    Authorization = "Basic $encodedAuthString"
}

Invoke-WebRequest -Method Put -Uri "https://blog.lindexi.com/artifactory/doubi.exe" -Headers $Headers -InFile "E:\lindexi\doubi.exe"
```

感谢工具人 [lsj](https://blog.sdlsj.net/ ) 提供的方法






<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。