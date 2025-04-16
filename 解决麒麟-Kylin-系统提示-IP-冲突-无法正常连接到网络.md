
# 解决麒麟 Kylin 系统提示 IP 冲突 无法正常连接到网络

麒麟 Kylin 系统插入网线但没有网，通知中心里面有网络设置工具的网络提示消息，消息内容是 IP 冲突，无法正常连接到网络

<!--more-->


<!-- CreateTime:2025/04/16 07:24:21 -->

<!-- 发布 -->
<!-- 博客 -->

我遇到的问题是 MAC 地址冲突导致的 IP 地址冲突，解决方法很简单，只需先进入到设置里面，先禁用有线网络，然后再执行以下命令，修改 MAC 地址

```
sudo ifconfig eth0 down
sudo ifconfig eth0 hw ether 1E:ED:19:25:1A:B2
sudo ifconfig eth0 up
```

以上命令行中，请将 eth0 换成你自己的有线网卡名，以上的 `1E:ED:19:25:1A:B2` 是我编造的 MAC 地址，还请自己随意编辑末尾几个数字。如果自己编造得不对，则会给出 无法指定被请求的地址 错误，继续更改即可

执行完成之后，重新启用有线网络即可




<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。