
# dotnet OpenXML 读取 PPT 动画进入退出强调动画类型

本文告诉大家，如何判断 PPT 的某个元素动画属于进入或退出或强调等类型的动画

<!--more-->


<!-- 发布 -->

根据 ECMA-376 文档可以了解到，在 PPT 动画中，通过 `cTn` 也就是 OpenXML sdk 定义的 `CommonTimeNode` 类型的 PresetClass 属性，即可用来判断当前的动画类型

例如新建一个空白的 PPT 文件，在里面放一个元素，然后设置飞入动画，此时的飞入动画是进入动画。通过 [解压缩文档为文件夹工具](https://blog.lindexi.com/post/dotnet-OpenXML-%E8%A7%A3%E5%8E%8B%E7%BC%A9%E6%96%87%E6%A1%A3%E4%B8%BA%E6%96%87%E4%BB%B6%E5%A4%B9%E5%B7%A5%E5%85%B7.html ) 解压缩此文件，可以看到在 Slide1.xml 有如下代码

```xml
<p:cTn id="5" presetID="2" presetClass="entr" presetSubtype="4" fill="hold" grpId="0" nodeType="clickEffect">
```

可以使用如下代码进行读取判断

```csharp
            using var presentationDocument =
                DocumentFormat.OpenXml.Packaging.PresentationDocument.Open("Test.pptx", false);
            var presentationPart = presentationDocument.PresentationPart;
            var slidePart = presentationPart!.SlideParts.First();
            var slide = slidePart.Slide;
            var timing = slide.Timing;
            // 第一级里面默认只有一项
            var commonTimeNode = timing?.TimeNodeList?.ParallelTimeNode?.CommonTimeNode;

            if (commonTimeNode?.NodeType?.Value == TimeNodeValues.TmingRoot)
            {
                // 这是符合约定
                // nodeType="tmRoot"
            }

            if (commonTimeNode?.ChildTimeNodeList == null) return;
            // 理论上只有一项，而且一定是 SequenceTimeNode 类型
            var sequenceTimeNode = commonTimeNode.ChildTimeNodeList.GetFirstChild<SequenceTimeNode>();

            var mainSequenceTimeNode = sequenceTimeNode.CommonTimeNode;
            if (mainSequenceTimeNode?.NodeType?.Value == TimeNodeValues.MainSequence)
            {
                // [TimeLine 对象 (PowerPoint) | Microsoft Docs](https://docs.microsoft.com/zh-cn/office/vba/api/PowerPoint.TimeLine )
                //  MainSequence 主动画序列
                var mainParallelTimeNode = mainSequenceTimeNode.ChildTimeNodeList
                    .GetFirstChild<ParallelTimeNode>().CommonTimeNode.ChildTimeNodeList
                    .GetFirstChild<ParallelTimeNode>().CommonTimeNode.ChildTimeNodeList
                    .GetFirstChild<ParallelTimeNode>();

                switch (mainParallelTimeNode.CommonTimeNode.PresetClass.Value)
                {
                    case TimeNodePresetClassValues.Entrance:
                        // 进入动画
                        break;
                    case TimeNodePresetClassValues.Exit:
                        // 退出动画
                        break;
                    case TimeNodePresetClassValues.Emphasis:
                        // 强调动画
                        break;
                    case TimeNodePresetClassValues.Path:
                        // 路由动画
                        break;
                    case TimeNodePresetClassValues.Verb:
                        break;
                    case TimeNodePresetClassValues.MediaCall:
                        // 播放动画
                        break;
                    default:
                        throw new ArgumentOutOfRangeException();
                }
            }
```

本文所有代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/f7c152959666fe9b6d543834fcb30a7ff6cf7e15/PptxDemo) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/f7c152959666fe9b6d543834fcb30a7ff6cf7e15/PptxDemo) 欢迎访问

可以通过如下方式获取本文的源代码，先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin f7c152959666fe9b6d543834fcb30a7ff6cf7e15
```

以上使用的是 gitee 的源，如果 gitee 不能访问，请替换为 github 的源

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
```

获取代码之后，进入 PptxDemo 文件夹

如需获取强调的课件，请使用 git 切换到 b8092ac9d12f315ae94ee2e53c3b4748d866b31b 这个 commit 即可拿到 Test.pptx 带强调动画，内容大概如下

```xml
<p:cTn id="5" presetID="25" presetClass="emph" presetSubtype="0" fill="hold" grpId="0" nodeType="clickEffect">
```

如需获取路径动画的课件，请使用 git 切换到 08c7d7a13b19cfa120b7a9971f88da5af96b4c75 这个 commit 即可拿到 Test.pptx 带路径动画，路径动画的内容大概如下

```xml
<p:cTn id="5" presetID="42" presetClass="path" presetSubtype="0" accel="50000" decel="50000" fill="hold" grpId="0" nodeType="clickEffect">
  <p:stCondLst>
    <p:cond delay="0" />
  </p:stCondLst>
  <p:childTnLst>
    <p:animMotion origin="layout" path="M 0 0 L 0 0.25 E" pathEditMode="relative" ptsTypes="">
      <p:cBhvr>
        <p:cTn id="6" dur="2000" fill="hold" />
        <p:tgtEl>
          <p:spTgt spid="2" />
        </p:tgtEl>
        <p:attrNameLst>
          <p:attrName>ppt_x</p:attrName>
          <p:attrName>ppt_y</p:attrName>
        </p:attrNameLst>
      </p:cBhvr>
    </p:animMotion>
  </p:childTnLst>
</p:cTn>
```

如需获取退出动画的课件，请使用 git 切换到 1521fdf2c9c94f13efe06dea25572e18847c11f3 这个 commit 即可拿到 Test.pptx 带退出动画，退出动画的内容大概如下

```xml
<p:cTn id="5" presetID="16" presetClass="exit" presetSubtype="21" fill="hold" grpId="0" nodeType="clickEffect">
  <p:stCondLst>
    <p:cond delay="0" />
  </p:stCondLst>
  <p:childTnLst>
    <p:animEffect transition="out" filter="barn(inVertical)">
      <p:cBhvr>
        <p:cTn id="6" dur="500" />
        <p:tgtEl>
          <p:spTgt spid="2" />
        </p:tgtEl>
      </p:cBhvr>
    </p:animEffect>
    <p:set>
      <p:cBhvr>
        <p:cTn id="7" dur="1" fill="hold">
          <p:stCondLst>
            <p:cond delay="499" />
          </p:stCondLst>
        </p:cTn>
        <p:tgtEl>
          <p:spTgt spid="2" />
        </p:tgtEl>
        <p:attrNameLst>
          <p:attrName>style.visibility</p:attrName>
        </p:attrNameLst>
      </p:cBhvr>
      <p:to>
        <p:strVal val="hidden" />
      </p:to>
    </p:set>
  </p:childTnLst>
</p:cTn>
```

如需获取多媒体动画的课件，请使用 git 切换到 4338d948558f7748c865b47672d4a579dea8353c 这个 commit 即可拿到 Test.pptx 带播放视频动画，内容大概如下

```xml
<p:par>
  <p:cTn id="5" presetID="1" presetClass="mediacall" presetSubtype="0" fill="hold" nodeType="clickEffect">
    <p:stCondLst>
      <p:cond delay="0" />
    </p:stCondLst>
    <p:childTnLst>
      <p:cmd type="call" cmd="playFrom(0.0)">
        <p:cBhvr>
          <p:cTn id="6" dur="1137" fill="hold" />
          <p:tgtEl>
            <p:spTgt spid="3" />
          </p:tgtEl>
        </p:cBhvr>
      </p:cmd>
    </p:childTnLst>
  </p:cTn>
</p:par>            
```

本文的属性是依靠 [dotnet OpenXML 解压缩文档为文件夹工具](https://blog.lindexi.com/post/dotnet-OpenXML-%E8%A7%A3%E5%8E%8B%E7%BC%A9%E6%96%87%E6%A1%A3%E4%B8%BA%E6%96%87%E4%BB%B6%E5%A4%B9%E5%B7%A5%E5%85%B7.html ) 工具协助测试的，这个工具是开源免费的工具，欢迎使用

更多请看 [Office 使用 OpenXML SDK 解析文档博客目录](https://blog.lindexi.com/post/Office-%E4%BD%BF%E7%94%A8-OpenXML-SDK-%E8%A7%A3%E6%9E%90%E6%96%87%E6%A1%A3%E5%8D%9A%E5%AE%A2%E7%9B%AE%E5%BD%95.html )





<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。