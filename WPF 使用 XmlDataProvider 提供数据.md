# WPF 使用 XmlDataProvider 提供数据

有一些界面逻辑包含了列表，列表里面的内容是咱可以预设置进去的，但是列表里面的内容是复杂的内容。此时我推荐使用 XmlDataProvider 提供数据，使用 XmlDataProvider 可以将所有逻辑都放在 XAML 上，十分清真

<!--more-->
<!-- CreateTime:2021/3/5 8:38:21 -->

<!-- 发布 -->

是否有小伙伴好奇 Binding 的 XPath 是用在哪里的？其实在使用 XmlDataProvider 提供数据，绑定就需要用到 XPath 属性，例如我需要定义书籍列表，书籍信息包括了 ISBN 序列号以及书名等信息，此时的定义可以是如下

```xml
        <Grid.Resources>
            <XmlDataProvider x:Key="InventoryData" XPath="Books">
                <x:XData>
                    <Books xmlns="">
                        <Book ISBN="0-7356-0562-9" Stock="in" Number="9">
                            <Title>XML in Action</Title>
                            <Summary>XML Web Technology</Summary>
                        </Book>
                        <Book ISBN="0-7356-1370-2" Stock="in" Number="8">
                            <Title>Programming Microsoft Windows With C#</Title>
                            <Summary>C# Programming using the .NET Framework</Summary>
                        </Book>
                        <Book ISBN="0-7356-1288-9" Stock="out" Number="7">
                            <Title>Inside C#</Title>
                            <Summary>C# Language Programming</Summary>
                        </Book>
                        <Book ISBN="0-7356-1377-X" Stock="in" Number="5">
                            <Title>Introducing Microsoft .NET</Title>
                            <Summary>Overview of .NET Technology</Summary>
                        </Book>
                        <Book ISBN="0-7356-1448-2" Stock="out" Number="4">
                            <Title>Microsoft C# Language Specifications</Title>
                            <Summary>The C# language definition</Summary>
                        </Book>
                    </Books>
                </x:XData>
            </XmlDataProvider>
        </Grid.Resources>
```

使用上面定义的资源作为数据，可以使用绑定资源的写法，如下面代码

```xml
 <ListView ItemsSource="{Binding Source={StaticResource InventoryData}, XPath=Book}"/>
```

可以看到在上面定义的 XML 内容里面，有两个定义的写法，如 ISBN 和 Title 两个不同的方式的定义。如 ISBN 是属性，而 Title 可以视为子元素，在 XAML 绑定里面，需要对其区分，对于子元素来说，只需要写对应的名字即可。但是对于 XML 属性，就需要在属性名前面加 `@` 符号，如下面代码

```xml
        <ListView ItemsSource="{Binding Source={StaticResource InventoryData}, XPath=Book}">
            <ListView.ItemsPanel>
                <ItemsPanelTemplate>
                    <StackPanel Orientation="Horizontal" />
                </ItemsPanelTemplate>
            </ListView.ItemsPanel>

            <ListView.ItemTemplate>
                <DataTemplate>
                    <Grid Background="#5a5a5a" Margin="10,10,10,10">
                        <StackPanel Margin="2,2,2,2">
                            <TextBlock Text="{Binding XPath=Title}" />
                            <TextBlock Text="{Binding XPath=@ISBN}" />
                        </StackPanel>
                    </Grid>
                </DataTemplate>
            </ListView.ItemTemplate>
        </ListView>
```

可以看到用上面方式编写的逻辑还是十分清真的

本文代码放在 [github](https://github.com/lindexi/lindexi_gd/blob/68cefabd097bf2f4fc35e3384f34e1dc622a67ad/PotrallTiscawMouger/PotrallTiscawMouger/MainWindow.xaml) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/blob/68cefabd097bf2f4fc35e3384f34e1dc622a67ad/PotrallTiscawMouger/PotrallTiscawMouger/MainWindow.xaml) 欢迎小伙伴访问

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
