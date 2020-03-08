# win10 edge 打开闪退问题

本文记录我找网上找到 edge 打不开问题的记录和修复方法

<!--more-->
<!-- CreateTime:2019/8/15 8:53:22 -->

<!-- csdn -->

通过系统日志和windows上报信息，可以从网上收到相同的日志，解决方案都是进行更新

[Web application crashes only in edge - Microsoft Edge Development](https://developer.microsoft.com/en-us/microsoft-edge/platform/issues/18976373/ )

[Microsoft Edge Crashes](https://social.technet.microsoft.com/Forums/en-US/5a7517ef-4e0e-44ca-994b-4db0c2fbd0bd/microsoft-edge-crashes?forum=win10itprogeneral )

系统日志信息是

```csharp

Version=1
EventType=MoAppHang
EventTime=132097464135465240
ReportType=3
Consent=1
UploadTime=132097464138072184
ReportStatus=268435456
ReportIdentifier=a0b96850-10a3-4724-a8e8-36d7601aa12d
IntegratorReportIdentifier=c2259f29-8d72-421f-83e5-f4e77e11a303
Wow64Host=34404
NsAppName=praid:MicrosoftEdge
AppSessionGuid=000036a4-0006-002e-ea5e-b9c0ef4dd501
TargetAppId=U:Microsoft.MicrosoftEdge_40.15063.674.0_neutral__8wekyb3d8bbwe!MicrosoftEdge
TargetAppVer=40.15063.674.0_neutral_!2018//10//10:07:30:22!b13cc3!MicrosoftEdge.exe
BootId=4294967295
TargetAsId=1989
Response.BucketId=5cb53a2b0bdef924e20a74a0f9ff0353
Response.BucketTable=5
Response.LegacyBucketId=1299979677187638099
Response.type=4
Sig[0].Name=程序包全名
Sig[0].Value=Microsoft.MicrosoftEdge_40.15063.674.0_neutral__8wekyb3d8bbwe
Sig[1].Name=应用程序名
Sig[1].Value=praid:MicrosoftEdge
Sig[2].Name=应用程序版本
Sig[2].Value=11.0.15063.1418
Sig[3].Name=应用程序时间戳
Sig[3].Value=5bbdaa8e
Sig[4].Name=挂起签名
Sig[4].Value=3dd1
Sig[5].Name=挂起类型
Sig[5].Value=4194304
DynamicSig[1].Name=OS 版本
DynamicSig[1].Value=10.0.15063.2.0.0.256.48
DynamicSig[2].Name=区域设置 ID
DynamicSig[2].Value=2052
DynamicSig[22].Name=其他挂起签名 1
DynamicSig[22].Value=3dd1049251c31b5e12d78bf3aa87b8d1
DynamicSig[23].Name=其他挂起签名 2
DynamicSig[23].Value=eeba
DynamicSig[24].Name=其他挂起签名 3
DynamicSig[24].Value=eebacf08d9a95714bca58a97615e1b3a
DynamicSig[25].Name=其他挂起签名 4
DynamicSig[25].Value=3dd1
DynamicSig[26].Name=其他挂起签名 5
DynamicSig[26].Value=3dd1049251c31b5e12d78bf3aa87b8d1
DynamicSig[27].Name=其他挂起签名 6
DynamicSig[27].Value=eeba
DynamicSig[28].Name=其他挂起签名 7
DynamicSig[28].Value=eebacf08d9a95714bca58a97615e1b3a
UI[3]=Microsoft Edge 未响应
UI[4]=如果关闭该程序，你可能丢失信息。
UI[5]=关闭程序
UI[6]=关闭程序
State[0].Key=Transport.DoneStage1
State[0].Value=1
FriendlyEventName=已停止响应并且被关闭
ConsentKey=AppHangXProcB1
AppName=Microsoft.MicrosoftEdge_8wekyb3d8bbwe!MicrosoftEdge
AppPath=C:\Windows\SystemApps\Microsoft.MicrosoftEdge_8wekyb3d8bbwe\MicrosoftEdge.exe
ReportDescription=出现了一个问题，该问题导致了此程序停止与 Windows 进行交互。
NsPartner=windows
NsGroup=windows8
ApplicationIdentity=F681BC071C9ABB1B59BEA65918E11CC1
MetadataHash=-1427362016
```

使用 DebugView 可以看到下面信息

```csharp
sihost.exe	onecoreuap\base\appmodel\execmodel\modern\lifetimemanager\plmimpl.cpp(1443)\modernexecserver.dll!00007FFB8F0377C6: (caller: 00007FFB92F81B31) ReturnHr(387) tid(1e18) 80004001 尚未实现
```

可以尝试的修复方法如下

升级系统到 1903 的时候就没有这个问题

安装 [KB4499181](https://support.microsoft.com/en-us/help/4499181/windows-10-update-kb4499181 ) 之后，重启可修复，这个补丁大小是 1.01GB 但我只在一台设备上安装

使用 dism 扫描系统

```csharp
DISM.exe /Online /Cleanup-image /Scanhealth
DISM.exe /Online /Cleanup-image /Checkhealth
DISM.exe /Online /Cleanup-image /Restorehealth 
```

扫描系统注册库

```csharp
dism /online /Cleanup-Image /RestoreHealth & sfc /SCANNOW?& for %d in (%windir%\system32\*.dll) do %windir%\system32\regsvr32.exe /s %d
```

在扫描系统之后，如果依然打不开 edge 那么请尝试结束以下进程

- edge 进程

- ApplicationFrameHost.exe
- RuntimeBroker.exe
- browser_broken.exe
- smartscreen.exe

还有建议将环境上报给微软，如果此时可以打开反馈中心，那么请通过反馈中心反馈给微软

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
