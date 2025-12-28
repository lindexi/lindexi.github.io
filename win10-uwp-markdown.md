# win10 uwp Markdown

# 需求分析
<!--more-->
<!-- CreateTime:2019/9/2 12:57:38 -->


## 输入需求

### 用户可以输入标题

用户可以输入多级标题，包括一级标题，二级标题，以及其他标题。

输入标题方式可以使用快捷键，也可以手动输入“#”，一个“#” 表示一级标题，两个个“#” 表示二级标题，三个“#” 表示三级标题，其他表示多级标题。

输入快捷键要求判断，当前用户输入点之前是否存在文字，如果存在，那么输入两个回车后输入“#”，如果用户输入点之前是“#”，那么继续输入“#”，如果用户输入点之前是回车，那么输入“#”。

### 用户可以输入加粗

用户可以使用快捷键或直接输入“`**重点内容**`”加粗。

其中“`**重点内容**`”的内容是加粗，如果用户输入快捷键，那么判断用户是否有选择字符，如果存在用户选中字符，那么直接把“`**重点内容**`”中的 “重点内容”代换为用户选中的字符。如果用户不存在选中字符，那么输入“`****`”，把光标移动到中间。

### 用户可以输入斜体

用户可以使用快捷键或直接输入“`*重点内容*`”，其中“重点内容”是斜体。

如果用户输入快捷键，那么判断用户是否有选择字符，如果存在用户选中字符，那么直接把“`*重点内容*`”中的 “重点内容”代换为用户选中的字符。如果用户不存在选中字符，那么输入“`**`”，把光标移动到中间。

### 用户可以输入 图片

用户可以使用快捷键或直接输入“`![](url)`”，其中url为用户输入的图片URL。

用户可以通过直接粘贴图片。如果用户有选中字符，那么在`[]`输入用户选中字符。

用户可以通过拖入图片插入。

### 用户可以输入 引用

用户可以使用快捷键或直接输入“> ”，引用可以多级。

如果用户输入快捷键，那么判断用户是否有选择字符，如果存在用户选中字符，那么在选中字符之前输入“> ”。

判断用户插入之前一字符是不是回车，如果是回车，那么直接输入“> ”。如果不是，输入两个回车，输入“> ”。


### 用户可以输入 书签

用户可以使用快捷键或直接输入“`< !-- bm -->`”。

书签可以让用户直接跳到书签。

### 用户可以输入 列表

用户可以通过直接输入或快捷键输入列表，其中包括有序列表和无序列表。

#### 有序列表

用户可以通过直接输入或快捷键输入有序列表 “` 1. 列表内容`”

如果用户输入快捷键，那么判断用户是否有选择字符，如果存在用户选中字符，那么在选中字符之前输入“ 1. ”。

判断用户插入之前一字符是不是回车，如果是回车，那么直接输入“ 1. ”。如果不是，输入两个回车，输入“ 1. ”。

#### 无序列表

用户可以通过直接输入或快捷键输入有序列表 “` -  列表内容`”

如果用户输入快捷键，那么判断用户是否有选择字符，如果存在用户选中字符，那么在选中字符之前输入“ -  ”。

判断用户插入之前一字符是不是回车，如果是回车，那么直接输入“ -  ”。如果不是，输入两个回车，输入“ -  ”。

有序列表可以多级，依靠“- ”前的空格。

### 用户可以输入 超链接

用户可以使用快捷键或直接输入“`[](url)`，其中url为用户输入的URL。

如果用户有选中字符，那么在`[]`输入用户选中字符。

### 用户可以输入 代码

用户可以使用快捷键或直接输入“\`\`”，输入代码。如果用户需要输入多行代码，可以输入
“\`\`\` 回车 \`\`\`”。

如果用户使用快捷键，判断用户是否存在选中字符，如果存在，在代码中输入用户选中字符。

判断用户插入之前一字符是不是回车，如果是回车，那么直接输入“\`\`\` 回车 \`\`\`”，把光标移动到中间。如果不是回车，输入“\`\`”。


### 用户可以输入 公式

用户可以使用快捷键或直接输入“\$\$”，如果要输入多行公式，输入“\$\$ 回车 \$\$”。

如果用户使用快捷键，判断用户是否存在选中字符，如果存在，在公式中输入用户选中字符。

如果用户插入之前一字符是回车，那么输入“\$\$ 回车 \$\$”，如果不是，输入“\$\$”，把光标移到中间。

### 粘贴

#### 图片

如果用户粘贴的是图片，软件可以识别多种图片，如果用户有选择图片需要水印，那么处理图片。

图片如果过大，软件可以压缩图片。

软件自动把图片上传到云，用户可自己选择多个云。包括 九幽、七牛云，sm.ms，ituku.tk，建立云。

用户可以使用自己的云。

软件后台上传云，用户插入图片时，先把图片放到资源文件夹，然后上传。保存到资源文件夹的图片名为时间+随机数。

插入时，输入“`![](本地图片)<! -- GUID  -->`”，上传完成，自动把输入代换“`![](url)<! -- 本地  -->`”。

文章可以通过直接粘贴图片，插入。

插入的图片先放到本地，如果粘贴的内容是文件，那么直接复制文件到文章同目录的文件夹，文章的文件夹名称和文章一样。复制进去需要修改文件名称为源文件名称+日期+随机数。如果粘贴的内容是图片，那么把图片转换为文件，同样放到文件夹，文件名为日期+随机数。

在文本，插入图片格式为`![图片描述](url)< !--图片-->`，一开始的URL是图片的本地路径位置，然后注释的`图片`是`ID`，其中ID是随机生成，作用在上传图片完成代换。输入后台上传代码为`(本地文件)< !--图片-->`和图片文件。根据图片文件上传完成，代换`(URL)< !--本地文件-->`。如果URL不可用，可以重新上传。重新上传的代码是`(URL)< !--本地文件-->`，根据本地文件获取文件，然后上传，代换URL。

#### 网站

如果用户粘贴网站，自动输入“`[url](url)`”。

#### 文件

用户粘贴文件，自动使用图片处理，如果不是图片，那么直接放到本地资源，如果图片过大，不上传到云。

输入“`[文件名](文件)`”

### 复制

用户可以使用快捷键“ctrl+c”复制，如果用户存在选中字符，复制选中字符。如果用户不存在选中字符，复制一行。


### 撤销

用户可以点击撤销按钮或快捷键撤销，撤销保存用户本次打开所有修改。用户可以取消撤销。

撤销时，自动把光标移动到撤销文本。

## 文件

用户可以新建、删除、重命名文件。

用户可以导入文件。

### 新建

用户在输入新建文件标题后新建文件。文件名为用户输入的标题，去掉不能做文件名的字符。如果存在相同的文件名，那么加随机数。

如果用户选择Jekyll，那么自动在输入文件名加上日期。

新建文件默认添加“# 标题”然后加上回车

创建文件夹和文件名相同，文件的所有资源放在文件夹。

如果文件夹名重复，那么不需对文件夹创建。

### 导入文件

如果导入文件格式错误，自动显示文件，添加格式。




## 文件格式

开始是头部，头部可以是统一的文字，文字提供可代换值。



```xml

<! -- head -- >
头部信息
<! -- head -- >

```

然后是博客信息

博客信息有博客标题，创建日期，修改日期，标签，作者，文章和作者和创建时间、修改时间的md5

```xml
< !--blog-->
博客标题:

创建日期:

修改日期:

标签:

作者:

文章和作者和创建时间、修改时间的md5:

< !--blog-->

```

注意冒号是英文的冒号，标签用分行或逗号隔开

接着简介，文章简介

```xml
< !--summary-->

< !--summary-->

```

然后是文章内容

文章内容完成就是文章最后文本，文章最后文本也是统一的文字，所有文章都一样，文字提供可代换值。

总的文章看起来就是

```xml

< !--head-->
头部信息
< !--head-->

< !--blog-->
博客标题:
创建日期:
修改日期:
标签:
作者:
文章和作者和创建时间、修改时间的md5:
< !--blog-->

< !--summary-->

< !--summary-->

文章内容

文章最后文本

```

用户可以直接修改文章标题。

在创建文件，自动添加文章的创建时间。

给用户看的不存在上面的，只有博客标题，显示的是“# 博客标题”

## 文件类

| 类型|字段|
|--|--| 
| StorageFile | File 文件|
| String | Title 标题|
| DateTime | CreateTime 创建时间|
| DateTime | Time  修改时间|
| String   | Md5  文章和作者和创建时间、修改时间的md5|
| String[] |category 标签|
| String   | Author  作者|
| String   | Excerpt 摘要，去掉HTML|
| String   | Content 内容|
| String   | References 最后文字|
| String   | Catalog 目录|
| String   | Bm      书签|

## 保存

### 自动保存

在用户停止输入或输入时间超过用户设置自动保存时间，自动保存。

自动保存时锁住文件

### 用户保存

用户可以点击保存或使用快捷键。

用户保存后，重新设置自动保存。

如果用户在其他软件修改文件，自动对比Git得到修改，询问用户是否使用外部修改。

### 编码

保存的文件格式为上文件格式，自动填写修改日期，自动添加作者，添加末尾文字，计算Md5。

保存为UTf-8。


## 设置 

### 设置界面

 1. 用户可以设置界面显示 
 字体大小、字体颜色、背景颜色。用户可以设置背景为图片，用户可以设置图片透明度。

 1. 用户可以保存设置为本仓库，可以保存全局。

 1. 用户可以设置是否显示行号。

 1. 用户可以设置是否可以折叠。

 1. 用户可以设置是否显示多标签。

 1. 用户可以设置是否自动保存。

 1. 用户可以设置自动保存时间。

 1. 用户可以设置是否上传远程Git。

 1. 用户可以设置远程Git地址。

 1. 用户可以设置远程Git账号、密码、密钥。

 1. 用户可以设置是否自动显示Html内容。

 1. 用户可以设置显示界面设置栏。

 1. 用户可以设置是否自动后台更新。

 1. 用户可以设置是否可以安装第三方插件。

 1. 用户可以设置是否保存自动上传远程。

 1. 用户可以设置是否自动备份。

 1. 用户可以设置所有仓库的文件开头字符和结尾字符、作者。
  设置开头字符可以是仓库的文件，自动把整个文件放到开头位置。

 1. 用户可以设置Git提交信息。
  Git提交信息可以自动代换，提供 {Time} 当前时间。{Author} 作者。{Title} 标题。
  如果存在多个文件，那么直接有多个标题。

 1. 用户可以设置图片是否需要水印。

 1. 用户可以设置图片水印。

 1. 用户可以设置图片上传云。

 1. 用户可以设置图片上传云账号。

 1. 用户可以设置图片压缩。


### 快捷键

用户可以设置快捷键，可以自动对于快捷键。

快捷键可以保存设置为本仓库，可以保存全局。

快捷键设置为文本，用户可以自定义。

代码片设置为仓库，存在Snippet文件夹，后缀为`.snippet`

<table log-set-param="table_view" class="table-view log-set-param"><tr><th>输出后的效果</th><th>Markdown</th><th>快捷键</th></tr><tr><td>Bold</td><td>**text**</td><td>Ctrl/⌘ + B</td></tr><tr><td><i>Emphasize</i></td><td>*text*</td><td>Ctrl/⌘ + I</td></tr><tr><td>Strike-through</td><td>~~text~~</td><td>Ctrl + Alt + U</td></tr><tr><td>Link</td><td>[title](http://)</td><td>Ctrl/⌘ + K</td></tr><tr><td>Inline Code</td><td>`code`</td><td>Ctrl/⌘ + Shift + K</td></tr><tr><td>Image</td><td>![alt](http://)</td><td>Ctrl/⌘ + Shift + I</td></tr><tr><td>List</td><td>* item</td><td>Ctrl + L</td></tr><tr><td>Blockquote</td><td>&gt; quote</td><td>Ctrl + Q</td></tr><tr><td>H1</td><td># Heading</td><td><br/>　　</td></tr><tr><td>H2</td><td>## Heading</td><td>Ctrl/⌘ + H</td></tr><tr><td>H3</td><td>### Heading</td><td>Ctrl/⌘ + H (x2)</td></tr></table>

### 代码片

用户可以设置代码片，代码片为用户输入字符后，按Tab显示的一段代码。

用户可以自设置，代码片的内容，添加代码片。

代码片格式

```xml
< snippet>
	< content >< ! [CDATA[ 
   输入显示的代码，其中可以用{1}代表第一次光标位置，并可以设置变量，如{1：this}，所以设置为{1：this}的代码片会代换为输入的字符。
]]>< /content>
	< tabTrigger>输入字符，按Tab输入代码片< /tabTrigger>
< /snippet>
```

代码片可以设置为本仓库，可以保存全局。

代码片设置为仓库，存在Snippet文件夹，后缀为`.snippet`

用户可以设置代码片，如果代码片不可用，提示用户。

## 界面

打开软件进入仓库，自动检测是否仓库存在。

用户选择新建或进入仓库。

仓库后台读取文件，显示RingProgress。

读取文件和设置完，显示Read文件。

如果用户新建仓库，可以设置：

仓库名，仓库名不能为空。

仓库作者，如果没有设置，为全局作者。

远程Git，用户可以选择设置Git仓，可以设置多个，可以设置分支。

仓库图片，用户可以设置仓库图片，仓库图片存在image.png。

### 流畅

所以的操作都不可在UI线程。

用户输入到响应时间为100ms 最多。

后台自动把文件转为HTML显示，如果用户设置实时显示。

如果用户没有设置实时显示，可以通过按键后生成。

后台上传图片，上传完成通知界面。

后台保存。

### 高亮

使用标题和代码、图片等，高亮。

用户可以设置不同的高亮。

自动修改标题字体大小。

代码可以根据代码高亮。

 - C# 代码

 - java 代码 

### 菜单

如果界面小，那么显示少的菜单，显示菜单可以用户自定义。

用户可以设置菜单的快捷键。

#### 目录

用户可以点击目录按钮或快捷键打开文件目录，在用户输入标题后，自动添加目录。

显示的目录可以有多级。

用户点击目录可以跳到标题的所在。

### 操作

#### 查找

用户可以查找文本是否存在打开的文件或整个仓库。

用户可以查找仓库的文件是否存在输入的文件名。

用户可以查找仓库的文件是否存在输入的标题。

用户可以通过设置是否大小写，设置正则查找。

查找得到可以通过点击直接显示对应文件。

#### 替换

用户可以替换打开的文件内容，用户可以替换仓库所有文件内容，用户可以使用正则得到可以替换内容。

替换完成报告替换处。

#### 选中

用户可以通过快捷键选中，用户选择可以从字到句子。

用户可以用快捷键全选，可以点击菜单全选。

### 分享

用户可以通过分享按钮，把文本分享。

分享可以分享QQ空间、微博、Github、分享应用。

分享内容可以是文件，可以是Html、pdf。

用户可以通过分享按钮，分享仓库，可以分享仓库文件，可以分享仓库做出Html、pdf。

用户后台分享，如果分享Html或pdf，后台生成。

### 折叠

用户可以折叠代码和标题对于的文本。

用户可以展开折叠代码。

## 拼写检查

用户可以使用拼写检查，自动检测用户输入，并提示用户可以使用的输入。

拼写检查支持英文和汉字，用户可以添加自定义词语到词库。

## 云

用户可以重新传图片到云。

用户可以选择整个文件夹的文件，把文件的图片上传到云。

自动显示云空间剩余，提醒用户不足。

用户可以使用私有账号，可以使用公有账号，提供公有账号让用户购买

云提供CDN加速，可以让用户购买。

### 上传

上传文件，如果使用私有云，自动新建文件夹，在上传后缀添加用户标识。

如果使用公有云，在文件后缀自动添加用户标识。

如果云不支持类型上传，寻找另一个云，把用户文件上传。如果所有云不支持，放本地。


## 创建Html 

如果用户设置实时，那么在用户更改文件，自动转换为Html，显示。

如果用户没设置实时，在用户点击显示，后台转换。

当用户分享时，自动转换分享。

转换的Html可以使用用户模板，用户可以设置全局模板，可以设置仓库模板，设置放在`layout`文件夹。

用户可以设置转换规则。

### 过滤器

<table> 
   <thead> 
    <tr> 
     <th>描述</th> 
     <th> <span class="filter">过滤器</span> 和 <span class="output">输出</span> </th> 
    </tr> 
   </thead> 
   <tbody> 
    <tr> 
     <td> <p class="name"><strong>日期转化为 XML 模式</strong></p> <p>将日期转化为 XML 模式 (ISO 8601) 的格式。</p> </td> 
     <td class="align-center"> <p> <code class="filter">{ { site.time | date_to_xmlschema } }</code> </p> <p> <code class="output">2008-11-17T13:07:54-08:00</code> </p> </td> 
    </tr> 
    <tr> 
     <td> <p class="name"><strong>日期转化为 RFC-822 格式</strong></p> <p>将日期转化为 RFC-822 格式，用于 RSS 订阅。</p> </td> 
     <td class="align-center"> <p> <code class="filter">{ { site.time | date_to_rfc822 } }</code> </p> <p> <code class="output">Mon, 17 Nov 2008 13:07:54 -0800</code> </p> </td> 
    </tr> 
    <tr> 
     <td> <p class="name"><strong>日期转化为短格式</strong></p> <p>将日期转化为短格式。</p> </td> 
     <td class="align-center"> <p> <code class="filter">{ { site.time | date_to_string } }</code> </p> <p> <code class="output">17 Nov 2008</code> </p> </td> 
    </tr> 
    <tr> 
     <td> <p class="name"><strong>日期转化为长格式</strong></p> <p>将日期转化为长格式。</p> </td> 
     <td class="align-center"> <p> <code class="filter">{ { site.time | date_to_long_string } }</code> </p> <p> <code class="output">17 November 2008</code> </p> </td> 
    </tr> 
    <tr> 
     <td> <p class="name"><strong>检索</strong></p> <p>选取键值对应的所有对象，返回一个数组。</p> </td> 
     <td class="align-center"> <p> <code class="filter">{ { site.members | where:&quot;graduation_year&quot;,&quot;2014&quot; } }</code> </p> </td> 
    </tr> 
    <tr> 
     <td> <p class="name"><strong>判断</strong></p> <p>选取表达式正确的所有对象，返回一个数组。</p> </td> 
     <td class="align-center"> <p> <code class="filter">{ { site.members | where_exp:&quot;item&quot;, &quot;item.graduation_year == 2014&quot; } }</code> <code class="filter">{ { site.members | where_exp:&quot;item&quot;, &quot;item.graduation_year &lt; 2014&quot; } }</code> <code class="filter">{ { site.members | where_exp:&quot;item&quot;, &quot;item.projects contains 'foo'&quot; } }</code> </p> </td> 
    </tr> 
    <tr> 
     <td> <p class="name"><strong>分组</strong></p> <p>根据所给属性将对象分组，返回一个数组。</p> </td> 
     <td class="align-center"> <p> <code class="filter">{ { site.members | group_by:&quot;graduation_year&quot; } }</code> </p> <p> <code class="output">[{&quot;name&quot;=&gt;&quot;2013&quot;, &quot;items&quot;=&gt;[...]},<br />{&quot;name&quot;=&gt;&quot;2014&quot;, &quot;items&quot;=&gt;[...]}]</code> </p> </td> 
    </tr> 
    <tr> 
     <td> <p class="name"><strong>XML 转码</strong></p> <p>对一些字符串转码，已方便显示在 XML 。</p> </td> 
     <td class="align-center"> <p> <code class="filter">{ { page.content | xml_escape } }</code> </p> </td> 
    </tr> 
    <tr> 
     <td> <p class="name"><strong>CGI 转码</strong></p> <p> CGI 转码，用于 URL 中，将所有的特殊字符转化为 %XX 的形式。 </p> </td> 
     <td class="align-center"> <p> <code class="filter">{ { “foo,bar;baz?” | cgi_escape } }</code> </p> <p> <code class="output">foo%2Cbar%3Bbaz%3F</code> </p> </td> 
    </tr> 
    <tr> 
     <td> <p class="name"><strong>URI 转码</strong></p> <p> URI 转码。 </p> </td> 
     <td class="align-center"> <p> <code class="filter">{ { “'foo, bar \\baz?'” | uri_escape } }</code> </p> <p> <code class="output">foo,%20bar%20%5Cbaz?</code> </p> </td> 
    </tr> 
    <tr> 
     <td> <p class="name"><strong>统计字数</strong></p> <p>统计文章中的字数。</p> </td> 
     <td class="align-center"> <p> <code class="filter">{ { page.content | number_of_words } }</code> </p> <p> <code class="output">1337</code> </p> </td> 
    </tr> 
    <tr> 
     <td> <p class="name"><strong>数组转换为句子</strong></p> <p>将数组转换为句子，列举标签时尤其有用。</p> </td> 
     <td class="align-center"> <p> <code class="filter">{ { page.tags | array_to_sentence_string } }</code> </p> <p> <code class="output">foo, bar, and baz</code> </p> </td> 
    </tr> 
    <tr> 
     <td> <p class="name"><strong>Markdown 支持</strong></p> <p>将 Markdown 格式的字符串转换为 HTML 。</p> </td> 
     <td class="align-center"> <p> <code class="filter">{ { page.excerpt | markdownify } }</code> </p> </td> 
    </tr> 
    <tr> 
     <td> <p class="name"><strong>Sass / SCSS 转换</strong></p> <p>将 Sass / SCSS 格式的字符串转换为 CSS</p> </td> 
     <td class="align-center"> <p> <code class="filter">{ { some_scss | scssify } }</code> </p> <p> <code class="filter">{ { some_sass | sassify } }</code> </p> </td> 
    </tr> 
    <tr> 
     <td> <p class="name"><strong>Slugify</strong></p> <p>将字符串转换为小写字母 URL “slug”。详见下面的参数。</p> </td> 
     <td class="align-center"> <p> <code class="filter">{ { &quot;The _config.yml file&quot; | slugify } }</code> </p> <p> <code class="output">the-config-yml-file</code> </p> <p> <code class="filter">{ { &quot;The _config.yml file&quot; | slugify: 'pretty' } }</code> </p> <p> <code class="output">the-_config.yml-file</code> </p> </td> 
    </tr> 
    <tr> 
     <td> <p class="name"><strong>JSON 转换</strong></p> <p>将 Hash / 数组 格式的字符串转换为 JSON</p> </td> 
     <td class="align-center"> <p> <code class="filter">{ { site.data.projects | jsonify } }</code> </p> </td> 
    </tr> 
    <tr> 
     <td> <p class="name"><strong>排序</strong></p> <p>对数组排序，可选参数为：1.排序属性；2.顺序（正序或倒序）</p> </td> 
     <td class="align-center"> <p> <code class="filter">{ { page.tags | sort } }</code> </p> <p> <code class="filter">{ { site.posts | sort: 'author' } }</code> </p> <p> <code class="filter">{ { site.pages | sort: 'title', 'last' } }</code> </p> </td> 
    </tr> 
    <tr> 
     <td> <p class="name"><strong>样本</strong></p> <p>从数组中选取一个随意值。可选参数为：选取个数 </p> </td> 
     <td class="align-center"> <p> <code class="filter">{ { site.pages | sample } }</code> </p> <p> <code class="filter">{ { site.pages | sample:2 } }</code> </p> </td> 
    </tr> 
    <tr> 
     <td> <p class="name"><strong>数组筛选</strong></p> <p>从一个数组中 Push, pop, shift, and unshift 元素。</p> <p>这些命令对原数组是<strong>无影响的</strong>。它们不会改变数组本身，而是创建副本后，对副本进行操作。</p> </td> 
     <td class="align-center"> <p> <code class="filter">{ { page.tags | push: 'Spokane' } }</code> </p> <p> <code class="output">['Seattle', 'Tacoma', 'Spokane']</code> </p> <p> <code class="filter">{ { page.tags | pop } }</code> </p> <p> <code class="output">['Seattle']</code> </p> <p> <code class="filter">{ { page.tags | shift } }</code> </p> <p> <code class="output">['Tacoma']</code> </p> <p> <code class="filter">{ { page.tags | unshift: &quot;Olympia&quot; } }</code> </p> <p> <code class="output">['Olympia', 'Seattle', 'Tacoma']</code> </p> </td> 
    </tr> 
    <tr> 
     <td> <p class="name"><strong>Inspect</strong></p> <p>将对象转换为其字符串表示形式，用于调试</p> </td> 
     <td class="align-center"> <p> <code class="filter">{ { some_var | inspect } }</code> </p> </td> 
    </tr> 
   </tbody> 
</table>

### 转html

自动在html 写文件md5，如果文件md5没修改，不转换。

## 创建pdf

## Git

