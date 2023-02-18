
# 收集的 C 盘 Windows System32 文件夹里的文件

这是我收藏的 C:\Windows\System32 文件夹里面的各个文件的作用或描述。有一些是我猜测的，有一些是我从网上找的描述，只有很少一些是我了解的。如果有写错的写漏的，欢迎评论告诉我或发邮件告诉我。本文非权威文档，请不要拿着本文的内容作为任何辩论的参考

<!--more-->


<!-- 草稿 -->

aadauthhelper.dll : Microsoft AAD（Azure Active Directory） Auth Helper 据说是 Microsoft Office Access 2010 的依赖文件，也属于 Office 系列依赖文件

aadcloudap.dll : 官方描述是 `AAD（Azure Active Directory） Cloud AP Plugin` 据说是 Microsoft Office Access 2010 的依赖文件，也属于 Office 系列依赖文件

aadjcsp.dll : 不知道是啥，猜测是 Office 系列依赖文件，也是 AAD （Azure Active Directory）系列文件

aadtb.dll : 官方描述是 `AADAzure Active Directory） Token Broker Helper Library` 据说是 Microsoft Office Access 2010 的依赖文件，也属于 Office 系列依赖文件

aadWamExtension.dll : 也是 AAD （Azure Active Directory）系列文件。不知道是啥，猜测是 Office 系列依赖文件

AAarSvc.dll : 这是 Agent Activation Runtime_70f961 服务，默认执行命令是 `C:\WINDOWS\system32\svchost.exe -k AarSvcGroup -p` 禁用方法请参阅 [官方文档](https://docs.microsoft.com/en-us/windows/application-management/per-user-services-in-windows)

AboutSettingsHandlers.dll : 官方描述是 `System Settings About Handlers Implementation` 翻译过来大概是 系统设置关于的实现

AboveLockAppHost.dll : 不知道是啥，猜测是 Office 系列依赖文件

accessibilitycpl.dll : 提供了一系列的无障碍辅助功能图标，此 DLL 最大的功能就是提供图标，详细请参阅 [Windows 10 自带那么多图标，去哪里找呢](https://blog.csdn.net/wpwalter/article/details/79394452) 博客。这是 Accessibility 系列文件，用来做无障碍辅助功能的，可以用来做辅助盲人聋哑人等使用计算机，也可以用来做 UI 自动化等，详细请看[官方文档](https://www.microsoft.com/en-us/accessibility/)

accountaccessor.dll : 官方描述是 `Sync data model to access accounts` 翻译过来大概是 同步数据模型以访问帐户

AccountsRt.dll : 官方描述是 `Accounts RT utilities for mail, contacts, calendar` 翻译过来大概是 邮件、联系人、日历的帐户 RT 实用程序

AcGenral.dll : Windows Compatibility DLL 不知道是啥

AcLayers.dll : AcLayers是windows在winXP引入的垫片机制的一个垫片，详细请看 [记一次win10下程序以兼容方式启动导致的死锁 - Development - Technology](https://conecoy.cn/Technology/Development/%E8%AE%B0%E4%B8%80%E6%AC%A1win10%E4%B8%8B%E7%A8%8B%E5%BA%8F%E5%85%BC%E5%AE%B9%E5%AF%BC%E8%87%B4%E7%9A%84%E6%AD%BB%E9%94%81/)

acledit.dll : 访问控制列表编辑器 

aclui.dll : 访问控制列表界面

acmigration.dll : Compatibility Upgrade Migration Host 兼容性升级迁移宿主

ACPBackgroundManagerPolicy.dll : ACP Background Manager Policy DLL

acppage.dll : 兼容性选项卡外壳扩展库

acproxy.dll : Autochk Proxy DLL

AcSpecfc.dll : Windows Compatibility DLL 详细请参阅 [https://github.com/strontic/strontic.github.io/blob/master/xcyclopedia/library/AcSpecfc.dll-BD6446D4FC9E5C01F3C48F20F241AC7B.md](https://github.com/strontic/strontic.github.io/blob/master/xcyclopedia/library/AcSpecfc.dll-BD6446D4FC9E5C01F3C48F20F241AC7B.md)

ActionCenter.dll : 操作中心，描述是 `Security and Maintenance` 安全与维护，预计是和控制面板里面的安全与维护有关

ActionCenterCPL.dll : 操作中心控制面板，预计就是控制面板里面的安全与维护功能

ActionQueue.dll : Unattend Action Queue Generator / Executor 无人值守的操作队列生成器或执行程序，详细请参阅 [https://github.com/strontic/strontic.github.io/blob/master/xcyclopedia/library/ActionQueue.dll-AAC8B18A373C9CE31D3E074A381D3A00.md](https://github.com/strontic/strontic.github.io/blob/master/xcyclopedia/library/ActionQueue.dll-AAC8B18A373C9CE31D3E074A381D3A00.md)

ActivationClient.dll : 激活的客户端，预计和激活相关

ActivationManager.dll : 激活的管理器，预计和激活相关

ActivationVdev.dll : 不知道是啥，预计和激活相关

activeds.dll : AD 路由器DLL据说没有它， 打开事件查看器会出错

activeds.tlb : 被 activeds.dll 依赖

ActiveSyncCsp.dll：详细请看 [https://learn.microsoft.com/en-us/windows/client-management/mdm/activesync-csp](https://learn.microsoft.com/en-us/windows/client-management/mdm/activesync-csp)


## 参考

[https://www.exefiles.com/](https://www.exefiles.com/)

[https://www.freefixer.com](https://www.freefixer.com)

[http://www.tceic.com/k06j50l32gl90242g995k547.html](http://www.tceic.com/k06j50l32gl90242g995k547.html)

[http://windows10dll.nirsoft.net](http://windows10dll.nirsoft.net)

[https://github.com/strontic/strontic.github.io](https://github.com/strontic/strontic.github.io)

[SYSTEM32 下的几乎所有文件的简单说明_AirZH??的博客-CSDN博客](https://blog.csdn.net/weixin_34290631/article/details/92284477)




<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。