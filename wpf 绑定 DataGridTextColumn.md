# wpf 绑定 DataGridTextColumn 

本文告诉大家如何在 DataGridTextColumn 使用绑定，因为很容易绑定就找不到数据。

<!--more-->
<!-- CreateTime:2018/8/10 19:16:53 -->


<!-- csdn -->

使用 DataGrid 有一个坑，就是 Columns 的数据绑定拿不到数据。

例如下面的代码

```csharp
   <Grid>
        <DataGrid x:Name="MyDataGrid" ItemsSource="{Binding DataList}" AutoGenerateColumns="False">
            <DataGrid.Columns>
                <DataGridTextColumn Header="Id" Binding="{Binding Id}"/>
                <DataGridTextColumn Header="Property1" Binding="{Binding Property1}"/>
                <DataGridTextColumn Header="Property2" Binding="{Binding Property2}"/>
          
                <DataGridTemplateColumn Header="Total">
                    <DataGridTemplateColumn.CellTemplate>
                        <DataTemplate>
                            <TextBlock>
                                <TextBlock.Text>
                                    <MultiBinding>
                                        <Binding Path="Property1"/>
                                        <Binding Path="Property2"/>
                                        <MultiBinding.Converter>
                                            <local:MyValueConverter/>
                                        </MultiBinding.Converter>
                                    </MultiBinding>
                                </TextBlock.Text>
                            </TextBlock>
                        </DataTemplate>
                    </DataGridTemplateColumn.CellTemplate>
                </DataGridTemplateColumn>
            </DataGrid.Columns>
        </DataGrid>
    </Grid>
```

可以看到绑定了`DataGridTextColumn Header="Property1" Binding="{Binding Property1}"` 可以拿到数值，但是在`MultiBinding`没有拿到数值，因为他在`DataTemplate`而`DataGridTemplateColumn`没有数据。

解决方法很简单，使用`RelativeSource`找到数据。只需要修改`<Binding Path="DataContext.Property2" RelativeSource="{RelativeSource AncestorType=DataGridCell}"/>` 就可以。下面就是修改后的代码

```csharp
  <Grid>
        <DataGrid x:Name="MyDataGrid" ItemsSource="{Binding DataList}" AutoGenerateColumns="False">
            <DataGrid.Columns>
                <DataGridTextColumn Header="Id" Binding="{Binding Id}"/>
                <DataGridTextColumn Header="Property1" Binding="{Binding Property1}"/>
                <DataGridTextColumn Header="Property2" Binding="{Binding Property2}"/>
                <DataGridTextColumn>
                    <DataGridTextColumn.Binding >
                        <MultiBinding >
                            <Binding Path="DataContext.Property1" RelativeSource="{RelativeSource AncestorType=DataGridCell}"/>
                            <Binding Path="DataContext.Property2" RelativeSource="{RelativeSource AncestorType=DataGridCell}"/>
                            <MultiBinding.Converter>
                                <local:MyValueConverter/>
                            </MultiBinding.Converter>
                        </MultiBinding>
                    </DataGridTextColumn.Binding>
                </DataGridTextColumn>
            </DataGrid.Columns>
        </DataGrid>
    </Grid>
```

![](http://image.acmx.xyz/34fdad35-5dfe-a75b-2b4b-8c5e313038e2%2F20171019115957.jpg)

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。 