
# dotnet OpenXML 解析 WPS 不规范的 PPT 文件的 cNvPr 重复 id 问题

在收到了反馈说有一份课件，打开解析就发现替换的元素不对，原因是这个课件里面的 Slide Master 里面存在一个元素的 id 和某个页面的元素 id 是相同的，这不符合 ECMA 376 的规范。通过读取文档的内容，发现这是 WPS 制作出来的 PPT 文件。本文做一个存档，用来告诉大家有这个坑

<!--more-->


<!-- CreateTime:2020/12/18 15:23:12 -->

<!-- 发布 -->

在 Office 2016 和 Office 2019 的行为判断请看以下两篇博客

[dotnet OpenXML 元素 cNvPr NonVisual Drawing Properties 的属性作用](https://blog.lindexi.com/post/dotnet-OpenXML-%E5%85%83%E7%B4%A0-cNvPr-NonVisual-Drawing-Properties-%E7%9A%84%E5%B1%9E%E6%80%A7%E4%BD%9C%E7%94%A8.html )

[dotnet OpenXML 元素 cNvPr NonVisual Drawing Properties 重复 id 标识处理](https://blog.lindexi.com/post/dotnet-OpenXML-%E5%85%83%E7%B4%A0-cNvPr-NonVisual-Drawing-Properties-%E9%87%8D%E5%A4%8D-id-%E6%A0%87%E8%AF%86%E5%A4%84%E7%90%86.html )

上面博客对于相同页面里面存在重复的 id 处理比较简单，但是对于在 Slide Master 里面存在一个元素的 id 和某个页面的元素 id 是相同的比较坑，但是做法就是将 Slide Master 里面存在相同 id 元素当成比较先发现的元素

下面咱来看看这份有趣的课件，测试课件请点击 [解析 WPS 不规范的 PPT 文件的 cNvPr 重复 id 问题.pptx](https://github.com/lindexi/lindexi_gd/blob/479eaa49de50116a3ea14641745e2b92fc51f4fb/%E6%96%87%E6%A1%A3/%E8%A7%A3%E6%9E%90%20WPS%20%E4%B8%8D%E8%A7%84%E8%8C%83%E7%9A%84%20PPT%20%E6%96%87%E4%BB%B6%E7%9A%84%20cNvPr%20%E9%87%8D%E5%A4%8D%20id%20%E9%97%AE%E9%A2%98.pptx) 下载

在这份课件的 SlideMaster1.xml 文件里面，可以看到有如下定义

```xml
      <p:sp>
        <p:nvSpPr>
          <p:cNvPr id="7" name="KSO_TEMPLATE" hidden="1" />
          <p:cNvSpPr />
          <p:nvPr userDrawn="1">
            <p:custDataLst>
              <p:tags r:id="rId8" />
            </p:custDataLst>
          </p:nvPr>
        </p:nvSpPr>
        <p:spPr>
          <a:xfrm>
            <a:off x="0" y="0" />
            <a:ext cx="0" cy="0" />
          </a:xfrm>
          <a:prstGeom prst="rect">
            <a:avLst />
          </a:prstGeom>
        </p:spPr>
        <!-- 忽略代码 -->
      </p:sp>
```

根据 [dotnet OpenXML 元素 cNvPr NonVisual Drawing Properties 的属性作用](https://blog.lindexi.com/post/dotnet-OpenXML-%E5%85%83%E7%B4%A0-cNvPr-NonVisual-Drawing-Properties-%E7%9A%84%E5%B1%9E%E6%80%A7%E4%BD%9C%E7%94%A8.html ) 可以了解到 `p:cNvPr` 的 id 就是元素的 id 属性

这个元素的属性是 `7` 同时有趣的是 `name="KSO_TEMPLATE"` 表示了这是 KSO 金山的 Template 模版

这个元素的 X 和 Y 和 宽度高度根据 `a:xfrm` 可以看到都是 0 没有宽度和高度，特别诡异

在页面 Slide1.xml 文件里面，可以看到里面有一个元素，这个元素的 `p:cNvPr` 也是 `7` 如下面代码

```xml
      <p:sp>
        <p:nvSpPr>
          <p:cNvPr id="7" name="标题 6" />
          <p:cNvSpPr txBox="1" />
          <p:nvPr>
            <p:custDataLst>
              <p:tags r:id="rId2" />
            </p:custDataLst>
          </p:nvPr>
        </p:nvSpPr>
        <!-- 忽略代码 -->
      </p:sp>
```

这个元素有一个有趣的属性，那就是 `p:custDataLst` 的值，这里面包含了 WPS 无文档的内容，可以看到 `p:tags` 通过 `r:id` 属性指向 `rId2` 的资源。在 `Slide1.xml.rels` 文件里面，可以看到下面代码

```xml
<Relationship Id="rId2" Type="http://schemas.openxmlformats.org/officeDocument/2006/relationships/tags" Target="../tags/tag12.xml"/>
```

也就是说加载了 tag12.xml 文件的内容，这个文件内容如下

```xml
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<p:tagLst xmlns:a="http://schemas.openxmlformats.org/drawingml/2006/main" xmlns:r="http://schemas.openxmlformats.org/officeDocument/2006/relationships" xmlns:p="http://schemas.openxmlformats.org/presentationml/2006/main">
  <p:tag name="KSO_WM_UNIT_HIGHLIGHT" val="0" />
  <p:tag name="KSO_WM_UNIT_COMPATIBLE" val="1" />
  <p:tag name="KSO_WM_UNIT_DIAGRAM_ISNUMVISUAL" val="0" />
  <p:tag name="KSO_WM_UNIT_DIAGRAM_ISREFERUNIT" val="0" />
  <p:tag name="KSO_WM_UNIT_LAYERLEVEL" val="1" />
  <p:tag name="KSO_WM_TAG_VERSION" val="1.0" />
  <p:tag name="KSO_WM_BEAUTIFY_FLAG" val="#wm#" />
  <p:tag name="KSO_WM_UNIT_ISCONTENTSTITLE" val="0" />
  <p:tag name="KSO_WM_UNIT_PRESET_TEXT" val="PART 01" />
  <p:tag name="KSO_WM_UNIT_NOCLEAR" val="0" />
  <p:tag name="KSO_WM_UNIT_VALUE" val="5" />
  <p:tag name="KSO_WM_UNIT_TYPE" val="e" />
  <p:tag name="KSO_WM_UNIT_INDEX" val="1" />
  <p:tag name="KSO_WM_TEMPLATE_CATEGORY" val="custom" />
  <p:tag name="KSO_WM_TEMPLATE_INDEX" val="20204217" />
  <p:tag name="KSO_WM_UNIT_ID" val="custom20204217_7*e*1" />
</p:tagLst>
```

可以看到这些都是 WPS 自己定义的内容，这样的课件在 Office 打开还是可以认的，只是还原效果没有在 WPS 上好。而在我自己的软件上，就凉凉了

这份课件可以通过 `docProps\custom.xml` 文件找到 WPS 应用的版本

```xml
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<Properties xmlns="http://schemas.openxmlformats.org/officeDocument/2006/custom-properties" xmlns:vt="http://schemas.openxmlformats.org/officeDocument/2006/docPropsVTypes">
  <property fmtid="{D5CDD505-2E9C-101B-9397-08002B2CF9AE}" pid="2" name="KSOProductBuildVer">
    <vt:lpwstr>2052-11.1.0.9198</vt:lpwstr>
  </property>
</Properties>
```

这里的 KSOProductBuildVer 就是表示构建这份文档的 WPS 应用版本，而 `2052-11.1.0.9198` 是版本号

本文的属性是依靠 [dotnet OpenXML 解压缩文档为文件夹工具](https://blog.lindexi.com/post/dotnet-OpenXML-%E8%A7%A3%E5%8E%8B%E7%BC%A9%E6%96%87%E6%A1%A3%E4%B8%BA%E6%96%87%E4%BB%B6%E5%A4%B9%E5%B7%A5%E5%85%B7.html ) 工具协助测试的，这个工具是开源免费的工具，欢迎小伙伴使用

更多请看 [Office 使用 OpenXML SDK 解析文档博客目录](https://blog.lindexi.com/post/Office-%E4%BD%BF%E7%94%A8-OpenXML-SDK-%E8%A7%A3%E6%9E%90%E6%96%87%E6%A1%A3%E5%8D%9A%E5%AE%A2%E7%9B%AE%E5%BD%95.html )





<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。