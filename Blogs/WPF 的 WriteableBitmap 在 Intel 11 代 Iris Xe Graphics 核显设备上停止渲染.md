在 Intel 11 代锐炬 Intel® Iris® Xe Graphics 核显设备上，如果此设备使用旧版本驱动，则可能导致 WPF 的 WriteableBitmap 停止渲染。此问题和 WPF 无关，此问题是 Intel 的 bug 且最新驱动版本已修复

<!--more-->


<!-- CreateTime:2024/1/13 9:36:19 -->

<!-- 发布 -->
<!-- 博客 -->

官方问题记录地址：<https://www.intel.cn/content/www/cn/zh/support/articles/000058136/graphics/graphics-for-11th-generation-intel-processors.html>

相关 WPF 记录：<https://github.com/dotnet/wpf/issues/3817>

解决方法，更新 Intel 显卡驱动，驱动版本大于等于 30.0.100.9667 可解此问题

影响显卡范围，以下按照 DeviceID 排列：

- 4C8A 为 i9-11900K 11900 11900T i7-11700K 11700 11700T i5-11600K 11600 11600T 11500 11500T 的核显 显示为 Intel(R) UHD Graphics 750
- 4C8B 为 i5-11400 11400T 的核显 显示为 Intel(R) UHD Graphics 730
- 9A78 为 i3-1125G4 1120G4 1115GRE 1115G4E 1115G4 1110G4 的核显 显示为 Intel(R) UHD Graphics
- 9A68 为 i5-11400H 11260H i3-11100HE 的核显 显示为 Intel(R) UHD Graphics
- 9A60 为 i9-11980HK 11950H 11900H i7-11850HE 11850H 11800H 11600H i5-11500HE 11500H 的核显 显示为 Intel(R) UHD Graphics
- 9A49 为 i7-11390H 11375H 11370H 1195G7 1185GRE 1185G7E 1185G7 1165G7 i5-11320H 11300H 1155G7 1145GRE 1145G7E 1145G7 1135G7 的核显 显示为 Intel(R) Iris(R) Xe Graphics
- 9A40 为 i7-1180G7 i5-1140G7 1130G7 的核显 显示为 Intel(R) Iris(R) Xe Graphics

由 [lsj](https://blog.sdlsj.net/ ) 帮忙编写识别 Intel 显卡类别代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/6f10958bbe27b2c288ac79da3f9c4600727c10a7/JayhallchocojejalNabinojajarher) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/6f10958bbe27b2c288ac79da3f9c4600727c10a7/JayhallchocojejalNabinojajarher) 上，可以使用如下方式获取

先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到由 [lsj](https://blog.sdlsj.net/ ) 帮忙编写识别 Intel 显卡类别代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin 6f10958bbe27b2c288ac79da3f9c4600727c10a7
```

以上使用的是 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin 6f10958bbe27b2c288ac79da3f9c4600727c10a7
```

获取代码之后，进入 JayhallchocojejalNabinojajarher 文件夹

此问题在 2022 的 3 月份就已经结束战斗，只是我之前以为我写过博客了，就一直没有写上。现在补上这篇博客

特别感谢：

- 感谢 [lsj](https://blog.sdlsj.net/ ) 帮忙编写识别 Intel 显卡类别代码以及测试 Intel i5-1135G7 使用 30.0.101.1404 版本驱动可以解决问题
- 感谢联想的伙伴们推进此问题
- 感谢 Intel 大佬们修复了此问题

关联博客：[WPF 的 Viewport3D 等 3D 模块在带 Intel UHD 770 设备上抛出渲染异常](https://blog.lindexi.com/post/WPF-%E7%9A%84-Viewport3D-%E7%AD%89-3D-%E6%A8%A1%E5%9D%97%E5%9C%A8%E5%B8%A6-Intel-UHD-770-%E8%AE%BE%E5%A4%87%E4%B8%8A%E6%8A%9B%E5%87%BA%E6%B8%B2%E6%9F%93%E5%BC%82%E5%B8%B8.html )
