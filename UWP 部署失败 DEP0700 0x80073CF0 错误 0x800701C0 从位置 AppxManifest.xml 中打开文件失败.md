# UWP 部署失败 DEP0700 0x80073CF0 错误 0x800701C0 从位置 AppxManifest.xml 中打开文件失败

本文记录 UWP 应用程序部署失败，提示 0x80073CF0 错误 0x800701C0: 从位置 AppxManifest.xml 中打开文件失败，错误为无法遍历该路径，因为它包含不受信任的装入点

<!--more-->
<!-- CreateTime:2025/01/17 07:05:23 -->

<!-- 发布 -->
<!-- 博客 -->

具体的错误内容如下

DEP0700: 应用程序注册失败。[0x80073CF0] 错误 0x800701C0: 从位置 AppxManifest.xml 中打开文件失败，错误为: 无法遍历该路径，因为它包含不受信任的装入点。 

对应的英文版错误大概内容如下

DEP0700: Registration of the app failed. [0x80073CF0] error 0x800701C0: Opening file from location: AppxManifest.xml failed with error: The path cannot be traversed because it contains an untrusted mount point.

我尝试删除 bin 和 obj 文件夹，重新生成都没有用。重新新建空白 UWP 项目也没有用

解决方法

此问题出现的本质原因是我的代码文件夹是一个放在使用 mklink 创建的软链接文件夹里面，软链接内的 AppxManifest.xml 文件不在受信任范围内，导致注册失败

知道了这个原因之后，我就去掉了软链接，换成实际的文件夹。不在软链接的文件夹内创建项目。如此即可解决此问题