# 关闭 SublimeText 3.2.2 Build 3211 的拼写检查

在更新 SublimeText 之后，发现所有的字符下面都有波浪线，这是新版本的 SublimeText 提供的拼写语法功能，然而我不需要此功能。本文来告诉大家如何关闭

<!--more-->
<!-- 发布 -->

点击标题栏的 Perferences 的 Settings 按钮，在 User 配置里面加上如下代码即可

```json
	"spell_check": false,
	"spelling_selector": "",
	"dictionary": ""
```

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
