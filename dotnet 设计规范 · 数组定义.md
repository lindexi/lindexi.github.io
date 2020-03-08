# dotnet 设计规范 · 数组定义

本文告诉大家数组定义需要知道的规范，本文翻译 [docs  dotnet](https://github.com/dotnet/docs/blob/master/docs/standard/design-guidelines/arrays.md )

<!--more-->
<!-- CreateTime:2018/7/9 14:26:48 -->

<!-- 标签：设计规范，规范 -->

✓ 建议在公开的 API 使用集合而不是数组。集合可以提供更多的信息。

X 不建议设置数组类型的字段为只读。虽然用户不能修改字段，但是可以修改字段里面的元素。如果需要一个只读的集合，建议定义为只读集合。

✓ 建议定义多维数组为一维，因为多维数组的性能比一维差。如果需要定义多维数组，请使用 `int[,] foo = new int[n, j]` ，请不要使用 `int[][] foo = new int[n][]`

参见：[docs/arrays.md at master · dotnet/docs](https://github.com/dotnet/docs/blob/master/docs/standard/design-guidelines/arrays.md )