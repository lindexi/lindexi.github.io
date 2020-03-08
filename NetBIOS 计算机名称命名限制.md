# NetBIOS 计算机名称命名限制

本文告诉大家对于 NetBIOS 的命名的限制

<!--more-->
<!-- CreateTime:2019/7/29 9:59:17 -->


## 长度限制

最小长度是 1 最长长度是 15 因为默认是 16 字符，但是微软使用最后一个字符作为后缀

## 可以使用的字符

可以使用英文和数字 

```csharp
abcdefghijklmnopqrstuvwxyz
ABCDEFGHIJKLMNOPQRSTUVWXYZ
0123456789
```

可以使用下面的符号

```
plus (+)
minus (-)
equals (=)
brackets ([ ])
comma (,)
tilde (~)
exclamation point (!)
at sign (@)
number sign (#)
dollar sign ($)
percent (%)
caret (^)
ampersand (&)
apostrophe (‘)
parentheses (())
braces ({})
underscore (_)
period (.)
```

不可以使用 `period (.)` 作为第一个字符，因为 `period (.)` 是用来做分段

```csharp
EGFCEFEECACACACACACACACACACACACA.NETBIOS.COM
```

## 不可以使用的字符

```
反斜杠	backslash (\)
正斜杠	slash mark (/)
冒号		colon (:)
星号		asterisk (*)
问号		question mark (?)
引号		quotation mark (")
小于号	less than sign (<) 
大于号	greater than sign (>)
竖线		vertical bar (|)
```

Microsoft Windows NT 中允许使用包含句点的非 DNS 名称。 但是，句点不能用于 Microsoft Windows 2000 或 Windows 的更新版本

## 保留字符

依照 RFC 952 的保留名称

```csharp
-GATEWAY
-GW
-TAC
```

[RFC 952 - DoD Internet host table specification](https://tools.ietf.org/html/rfc952 )

[Computer name conventions](http://ivan.dretvic.com/2012/10/computer-name-conventions/ )

[Active Directory user naming conventions](https://activedirectorypro.com/active-directory-user-naming-convention/ )

[MS-NBTE NetBIOS Name Syntax](https://msdn.microsoft.com/en-us/library/dd891456.aspx )

[Active Directory 中计算机、域、站点和 OU 的命名约定](https://support.microsoft.com/zh-cn/help/909264/naming-conventions-in-active-directory-for-computers-domains-sites-and )

[http://www.ietf.org/rfc/rfc1001.txt](http://www.ietf.org/rfc/rfc1001.txt )

[https://www.ietf.org/rfc/rfc1002.txt](https://www.ietf.org/rfc/rfc1002.txt )

[NetBIOS协议_百度百科](https://baike.baidu.com/item/NetBIOS%E5%8D%8F%E8%AE%AE )

关于文件的限制请看 [C# 不能用于文件名的字符](https://lindexi.gitee.io/post/C-%E4%B8%8D%E8%83%BD%E7%94%A8%E4%BA%8E%E6%96%87%E4%BB%B6%E5%90%8D%E7%9A%84%E5%AD%97%E7%AC%A6.html )

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
