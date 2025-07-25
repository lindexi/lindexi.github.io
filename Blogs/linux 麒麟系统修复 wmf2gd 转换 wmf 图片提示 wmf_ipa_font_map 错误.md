---
title: linux 麒麟系统修复 wmf2gd 转换 wmf 图片提示 wmf_ipa_font_map 错误
description: 本文记录在 2403 麒麟系统上，使用 wmf2gd 转换 wmf 图片时提示 ERROR: font.c (1334): wmf_ipa_font_map: failed to load any font! 错误问题
tags: 
category: 
---

<!-- CreateTime:2025/07/25 07:13:48 -->

<!-- 发布 -->
<!-- 博客 -->

以下是我的转换命令

```
./wmf2gd -t png -o 1.png image.wmf
```

报错信息如下

```
ERROR: font.c (1334): wmf_ipa_font_map: failed to load *any* font!
```

查阅本文下方的参考文档了解到，这是因为 wmf 尝试去读取 gsfonts 字体失败

我尝试使用 `ls /usr/share/fonts/type1` 命令了解到我当前的麒麟系统没有安装 gsfonts 字体

于是我就直接去 ubuntu 源下载 gsfonts 字体，下载代码如下

```
wget https://cn.archive.ubuntu.com/ubuntu/pool/universe/g/gsfonts/gsfonts_8.11%2Burwcyr1.0.7~pre44-4.5_all.deb
```

下载之后，使用如下代码进行安装

```
sudo dpkg -i gsfonts_8.11+urwcyr1.0.7~pre44-4.5_all.deb
```

安装之后即可正常使用 wmf2gd 转换 wmf 为 png 图片

这个过程中，我尝试用了 `--wmf-fontdir` 参数，发现是没有什么作用的，其命令如下

```
./wmf2gd -t png -o 1.png --wmf-fontdir=/usr/share/fonts/type1/gsfonts image.wmf
```

但如果自己将 gsfonts 解压出来，将 pfb 等字体放在自己的文件夹，再使用 `--wmf-fontdir` 设置到自己的文件夹，这是有用的

本文的所有方法对 wmf2svg 等工具同样生效

参考文档：

[ERROR: font.c (1334): wmf_ipa_font_map: failed to load *any* font! · Issue #25 · kakwa/libvisio2svg](https://github.com/kakwa/libvisio2svg/issues/25 )

[Convert WMF file fails - Legacy ImageMagick Discussions Archive](http://www.imagemagick.org/discourse-server/viewtopic.php?t=18987 )

[Bug #629153 “wmf2XXX tools fail due to defoma font issue” : Bugs : libwmf package : Ubuntu](https://bugs.launchpad.net/ubuntu/+source/libwmf/+bug/629153 )

[[solved] wmf2svg: failed to load *any* font - debianforum.de](https://debianforum.de/forum/viewtopic.php?t=122797 )

[gsfonts usage in ImageMagick - Legacy ImageMagick Discussions Archive](https://legacy.imagemagick.org/discourse-server/viewtopic.php?t=35933 )
