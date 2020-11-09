
# dotnet 使用 Obsolete 特性标记成员过时保持库和框架的兼容性

在开发库以及框架的时候，持续维护会遇到兼容性的问题，如发现了旧版本有一些接口设计不合理，或者方法命名不符合逻辑等。此时如果直接更改原有的属性名或方法名甚至类名等，将会导致上层业务的开发者们在升级库之后构建不通过，因为缺少对应的方法。此时就需要上层业务的开发者们查阅文档才能了解如何应对升级之后带来的变动

<!--more-->


<!-- 草稿 -->
<!-- 不发布 -->

在 dotnet 里面，可以使用 Obsolete 这个编译器分析辅助特性，给某个成员，如类和属性和方法事件等标记过时。这个 Obsolete 特性可以用来辅助库和框架开发者，在发生 API 变动时，可以保持兼容，或者提醒上层业务的开发者们如何应对

进行不兼容代码层 API 的变动，包括类名、属性名、方法名等所有公开的命名变更，以及命名空间的变更。还有



本文代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/d20149b7ace4d0b6e8ebb0a00aaede29a8de5118/BegibaberGawhilofigurwhal) 欢迎小伙伴访问






<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。