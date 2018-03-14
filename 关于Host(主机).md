
# 关于Host(主机)


<!--more-->



<div id="toc"></div>

|Host(主机)|用法|
|--|-|
|www|解析后的域名为 www.figotan.org|
|@|直接解析主域名 figotan.org|
|*|泛解析，匹配其他所有子域名 *.figotan.org|

## 关于Resord Type(记录类型)

要指向空间商提供的 IP 地址，选择「类型 A」，要指向一个域名，选择「类型 CNAME」

|Resord Type(记录类型)|用法|Points To|
|-|-|-|
|A(Host)|地址记录，用来指定域名的IPv4地址（如：8.8.8.8），如果需要将域名指向一个IP地址，就需要添加A记录|填写您服务器 IP，如果您不知道，请咨询您的空间商|
|AAAA(IPv6 Host)|用来指定主机名（或域名）对应的IPv6地址（例如：ff06:0:0:0:0:0:0:c3）记录|不常用。解析到 IPv6 的地址|
|CNAME(Alias)|如果需要将域名指向另一个域名，再由另一个域名提供ip地址，就需要添加CNAME记录|填写空间商给您提供的域名，例如：figotan.org|
|MX(Mail Exchanger)|如果需要设置邮箱，让邮箱能收到邮件，就需要添加MX记录|填写您邮件服务器的IP地址或企业邮局给您提供的域名，如果您不知道，请咨询您的邮件服务提供商|
|TXT(Text)|在这里可以填写任何东西，长度限制255。绝大多数的TXT记录是用来做SPF记录（反垃圾邮件）|一般用于 Google、QQ等企业邮箱的反垃圾邮件设置|
|SRV(Service)|记录了哪台计算机提供了哪个服务。格式为：服务的名字、点、协议的类型，例如：`_xmpp-server._tcp`|不常用。格式为：优先级、空格、权重、空格、端口、空格、主机名，记录生成后会自动在域名后面补一个“.”，这是正常现象。例如：5 0 5269 xmpp-server.l.google.com.|
|NS(Name Server)|域名服务器记录，如果需要把子域名交给其他DNS服务商解析，就需要添加NS记录|不常用。系统默认添加的两个NS记录请不要修改。NS向下授权，填写dns域名，例如：f1g1ns1.dnspod.net|

关于TTL

|TTL|用法|
|-|-|
|600（10分钟）|建议正常情况下使用 600|
|60（1分钟）|如果您经常修改IP，修改记录一分钟即可生效。长期使用 60，解析速度会略受影响|
|3600（1小时）|如果您IP极少变动（一年几次），建议选择 3600，解析速度快。如果要修改IP，提前一天改为 60，即可快速生效|

[http://www.figotan.org/2016/03/29/how-to-speed-up-your-blog-using-duplex-git-pages/](http://www.figotan.org/2016/03/29/how-to-speed-up-your-blog-using-duplex-git-pages/ )






<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。