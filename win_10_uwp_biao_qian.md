# win 10 UWP 标签

本文主要翻译：[http://visuallylocated.com/post/2015/02/20/Creating-a-WrapPanel-for-your-Windows-Runtime-apps.aspx](http://visuallylocated.com/post/2015/02/20/Creating-a-WrapPanel-for-your-Windows-Runtime-apps.aspx)    [http://depblog.weblogs.us/2015/02/18/how-to-add-a-tag-list-into-winrt-universal-apps/](http://depblog.weblogs.us/2015/02/18/how-to-add-a-tag-list-into-winrt-universal-apps/)  

我们需要给用户很多标签，我们需要使用一个控件，他的长度是变化，可以快速放，这样好像wrapPancel就是我们需要，因为这个我直接写如果看起来不懂，可以看
<!--more-->
<!-- CreateTime:2018/2/13 17:23:03 -->


<div id="toc"></div>

![这里写图片描述](http://img.blog.csdn.net/20160428154345998)

我们点添加就会添加标签，我们删除标签就很快排版。

我们使用RichBox，这个可以做我们标签

源代码因为作者写的和UWP不一样，我改UWP，放在https://github.com/lindexi/TagList

运行效果

![这里写图片描述](http://img.blog.csdn.net/20160429102218298)

点击按钮
![这里写图片描述](http://img.blog.csdn.net/20160429102248655)

删除
![这里写图片描述](http://img.blog.csdn.net/20160429102311111)

软件使用，先add
![这里写图片描述](http://img.blog.csdn.net/20160429102913148)

跳到让用户选择，这里如果让用户输入，使用有点难，可以使用用户在跳转输入，输入自动变为预设一样

```csharp
源.Add(new Tag() {Id = "id",Label = "用户输入"});
```

![这里写图片描述](http://img.blog.csdn.net/20160429103328037)

选择标签，选择完成保存

![这里写图片描述](http://img.blog.csdn.net/20160429103355334)

![这里写图片描述](http://img.blog.csdn.net/20160429103411522)

可以看到首页

![这里写图片描述](http://img.blog.csdn.net/20160429103435514)

标签使用在跳转MainPage

```csharp
if (e.NavigationMode == NavigationMode.Back)
```

我们把选择保存

```csharp
General.GetInstance().TagSelection
```

在`SetTags`是本算法的主要

我们搜索全部新加和被删除

```csharp
var tagParagraph = (Paragraph) (from paragraph in TagRichTextBlock.Blocks
    where paragraph.Name.StartsWith("Tags")
    select paragraph).FirstOrDefault();

var tagIds = from tag in General.GetInstance().TagSelection.Tags
    select tag.Id;

var buttonsToRemove = from item in tagParagraph.Inlines.Cast<InlineUIContainer>()
    where !tagIds.Contains(((Button) item.Child).Name)
    select item;

foreach (InlineUIContainer container in buttonsToRemove)
    tagParagraph.Inlines.Remove(container);
```

```csharp
                IEnumerable<string> buttonIds = from item in tagParagraph.Inlines.Cast<InlineUIContainer>()
                    select ((Button) item.Child).Name;

                IEnumerable<Tag> tagsToAdd = from item in General.GetInstance().TagSelection.Tags
                    where !buttonIds.Contains(item.Id)
                    select item;

                foreach (Tag tag in tagsToAdd)
                {
                    InlineUIContainer container = new InlineUIContainer();
                    RichTextBlock inlineRichTextBlock = new RichTextBlock()
                    {
                        IsTextSelectionEnabled = false
                    };
                    Paragraph inlineParagraph = new Paragraph();
                    inlineParagraph.Inlines.Add(new Run()
                    {
                        Text = string.Format("{0} ", tag.Label),
                        FontSize = 14
                    });
                    inlineParagraph.Inlines.Add(new Run()
                    {
                        Text = "\uE106",
                        FontFamily = new FontFamily("Segoe UI Symbol"),
                        FontSize = 10
                    });
                    inlineRichTextBlock.Blocks.Add(inlineParagraph);

                    Button tagButton = new Button()
                    {
                        Content = inlineRichTextBlock,
                        Style = (Style) Application.Current.Resources["TagButtonStyle"],
                        Name = tag.Id
                    };
                    tagButton.Click += OnTagButtonClicked;
                    container.Child = tagButton;

                    tagParagraph.Inlines.Add(container);
                }
```

点击删除按钮，删除id

```csharp
            string tagId = ((Button) sender).Name;
            General.GetInstance()
                .TagSelection.Tags.Remove(General.GetInstance().TagSelection.Tags.Single(item => item.Id.Equals(tagId)));
            SetTags();
```


源码：https://github.com/Depechie/TagList

