
# uwp ScrollViewer content out of panel when set the long width

I find a ununderstandable behavior in UWP. I make a content with a long width in ScrollViewer and the content will out of the panel.

<!--more-->


<!-- csdn -->

I write a xaml with a ScrollViewer and a Grid. And than I set a content to ScrollViewer and the content is a long width Grid.

```xaml
    <Grid>
        <Grid.ColumnDefinitions>
            <ColumnDefinition />
            <ColumnDefinition />
        </Grid.ColumnDefinitions>
        <ScrollViewer HorizontalScrollBarVisibility="Visible">
            <Grid Width="2100000">
                <TextBlock Margin="10,10,10,10"
                    Text="asdasdasdasdasdasdasdasdasdaasdasdasdasdasdasdasdasdasdasdasdasdasdasdaasdasdasdasdasdasdasdasdasdasdasdasdasdasdaasdasdasdasdasdasdasdasdasdasdasdasdasd" />
            </Grid>
        </ScrollViewer>
        <Grid Grid.Column="1" Background="#aa565656" />
    </Grid>
```

Running the code, and the TextBlock will out of the Grid.

After I set the Grid width to 2000000 that work well.

Do not ask why I can find it. I wrote a code `await "lindexi is doubi";` and my friend pressed my head to the Keyboard. My head pressed the `0` key and set the Grid width. I ran the code and I was surprised to see it.





<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。