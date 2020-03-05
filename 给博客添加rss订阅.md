# 给博客添加rss订阅

如果是自己搭建博客，有一个问题是如何写一篇新的文章就可以告诉读者，你写了一篇新的？一个简单方法是使用 rss ，RSS订阅是站点用来和其他站点之间共享内容的一种简易方式,即Really Simple Syndication(简易信息聚合)，使用这个东西就可以把自己写的博客推送给读者。

本文告诉大家，如果在博客配置 rss 让读者可以订阅。

<!--more-->
<!-- CreateTime:2019/9/2 12:57:38 -->


一般在很多网站都可以看到这个图标，这就是rss，他可以把最新的文章发给你。如果自己的博客没有添加 rss 那么大家需要打开博客才可以发现你写了新的文章，所以建议在博客添加。

![](http://image.acmx.xyz/34fdad35-5dfe-a75b-2b4b-8c5e313038e2%2F2017127131016.jpg)

![](http://image.acmx.xyz/34fdad35-5dfe-a75b-2b4b-8c5e313038e2%2F2017127131032.jpg)

在配置之前，需要知道 rss 的格式，实际上 rss 的格式可以看 [www.w3school.com.cn/rss/rss_syntax.asp](www.w3school.com.cn/rss/rss_syntax.asp) ，但是使用的很简单。

```csharp
<?xml version="1.0" encoding="UTF-8"?>  
<rss version="2.0">  
  <channel>  
    <title>博客标题</title>  
    <link>博客地址</link>  
    <description>网站描述</description>  
    <language>语言</language>  
    <item>  
      <title>第一篇</title>  
      <description>摘要</description>  
      <author>作者</author>  
      <pubDate>发布时间</pubDate>  
      <link>链接</link>  
      <guid>链接</guid>  
    </item>  
  <channel>  
</rss>  
```

可以看到 channel 的 title 指的是博客的标题，language 指的是语言，如中文 zh-CN 。如果有多个博客，那么写在 item 那里，其中 link 和 guid 设置为相同。

请看我博客的 rss ，atom 是可以不写

```csharp
<?xml version="1.0" encoding="UTF-8"?>
<rss version="2.0" xmlns:atom="http://www.w3.org/2005/Atom">
  <channel>
    <title>林德熙</title>
    <description>Windows 10 Developer</description>
    <link>https://lindexi.gitee.io/lindexi/</link>
    <atom:link href="https://lindexi.gitee.io/lindexi/feed.xml" rel="self" type="application/rss+xml"/>
    <pubDate>Thu, 07 Dec 2017 11:02:39 +0800</pubDate>
    <lastBuildDate>Thu, 07 Dec 2017 11:02:39 +0800</lastBuildDate>
    <generator>Jekyll v3.4.3</generator>
    <item>  
      <title>第一篇</title>  
      <description>摘要</description>  
      <author>作者</author>  
      <pubDate>发布时间</pubDate>  
      <link>链接</link>  
      <guid>链接</guid>  
    </item> 
    <item>  
      <title>第二篇</title>  
      <description>摘要</description>  
      <author>作者</author>  
      <pubDate>Wed, 29 Nov 2017 00:00:00 +0800</pubDate>
      <category>uwp</category>  
      <link>链接</link>  
      <guid>链接</guid>  
    </item> 
    </channel>
    </rss>
```

需要注意这里的时间都是这样写`Wed, 29 Nov 2017 00:00:00 +0800`，所以可以使用自动生成。如果使用的是 jekyll 大概可以直接复制我下面代码到 rss 文件。实际上去我希望你去我的项目复制文件。

{% raw %}

```csharp
<?xml version="1.0" encoding="UTF-8"?>
<rss version="2.0" xmlns:atom="http://www.w3.org/2005/Atom">
  <channel>
    <title>{{ site.title | xml_escape }}</title>
    <description>{{ site.description | xml_escape }}</description>
    <link>{{ site.url }}{{ site.baseurl }}/</link>
    <atom:link href="{{ "/feed.xml" | prepend: site.baseurl | prepend: site.url }}" rel="self" type="application/rss+xml"/>
    <pubDate>{{ site.time | date_to_rfc822 }}</pubDate>
    <lastBuildDate>{{ site.time | date_to_rfc822 }}</lastBuildDate>
    <generator>Jekyll v{{ jekyll.version }}</generator>
    {% for post in site.posts limit:10 %}
      <item>
        <title>{{ post.title | xml_escape }}</title>
        <description>{{ post.content | xml_escape }}</description>
        <pubDate>{{ post.date | date_to_rfc822 }}</pubDate>
        <link>{{ post.url | prepend: site.baseurl | prepend: site.url }}</link>
        <guid isPermaLink="true">{{ post.url | prepend: site.baseurl | prepend: site.url }}</guid>
        {% for tag in post.tags %}
        <category>{{ tag | xml_escape }}</category>
        {% endfor %}
        {% for cat in post.categories %}
        <category>{{ cat | xml_escape }}</category>
        {% endfor %}
      </item>
    {% endfor %}
  </channel>
</rss>

```
{% endraw %}

需要在 `_config.yml` 设置 site.description 等

现在很多小伙伴喜欢在极客头条分享自己的博客，这也是我访问最多的地方。

这样就可以写完博客告诉小伙伴，让大家学到。

推荐一个rss工具 [攸阅](http://hiiman.com/ ) 可以把自己订阅的博客发到邮箱。

如果需要自己写一个，可以抄袭我的[win10 UWP RSS阅读器 ](https://lindexi.github.io/lindexi/post/win10-UWP-RSS%E9%98%85%E8%AF%BB%E5%99%A8.html )

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。 