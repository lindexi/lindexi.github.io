# dotnet 获取网页里面 div 的方法

<!--more-->
<!-- 不发布 -->

通过正则获取

```
<div\sclass="a">
[^<>]*
(
  (
    (?'d'<div[^>]*>)
    [^<>]*
  )+
  (
    (?'-d'</div>)[^<>]*)+
  )*
  (?(d)(?!)
)
</div>
```

通过 Winista.HtmlParser 获取

