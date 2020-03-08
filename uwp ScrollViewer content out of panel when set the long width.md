# uwp ScrollViewer content out of panel when set the long width

I find a ununderstandable behavior in UWP. I make a content with a long width in ScrollViewer and the content will out of the panel.

<!--more-->
<!-- CreateTime:2019/3/15 14:14:24 -->

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

Code https://github.com/lindexi/lindexi_gd/tree/550373c7ee587980a7ecd56e2c03aa40c6972416/TucasurchasGeldiskaycasoohou