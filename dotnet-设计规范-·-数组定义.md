
# dotnet 设计规范 · 数组定义

本文告诉大家数组定义需要知道的规范，本文翻译 [docs  dotnet](https://github.com/dotnet/docs/blob/master/docs/standard/design-guidelines/arrays.md )

<!--more-->


<!-- CreateTime:2018/7/9 14:26:48 -->

<!-- 标签：设计规范，规范 -->

✓ 建议在公开的 API 使用集合而不是数组。集合可以提供更多的信息。

X 不建议设置数组类型的字段为只读。虽然用户不能修改字段，但是可以修改字段里面的元素。如果需要一个只读的集合，建议定义为只读集合。

✓ 建议定义多维数组为一维，因为多维数组的性能比一维差。如果需要定义多维数组，请使用 `int[,] foo = new int[n, j]` ，请不要使用 `int[][] foo = new int[n][]`

参见：[docs/arrays.md at master · dotnet/docs](https://github.com/dotnet/docs/blob/master/docs/standard/design-guidelines/arrays.md )




<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。