本文主要讲ListView等列表可以根据内容不同，使用不同模板的列表模板选择器，DataTemplateSelector。本文首发Win10.me，因为csdn总是被盗，上次维权失败，所以选择九幽首发，Voidcn也是，有我的连接，那些垃圾网站很都是修改文章不写出错。 

好啦，我们先来说下我们在什么下需要使用，其实就是当我们的数据有多样。

例如我们做了一个类，叫做 人，这时我们继承人做出来 男生 和女生，那么男生的属性可能和女生的不同

假设我们的 人有个属性叫做名称，而男生有个属性叫身高，女孩有个属性叫年龄，当然女生年龄放出来并不好，不过我也没找到别的。

好啦，我们在ViewModel放一个ObservableCollection&lt;Human&gt; HumanWord,这是我们发现，在前台不好弄，如何让列表显示男生和女孩，因为他们的属性不同。

这时就需要我们做选择器，这个可以根据我们传入选择模板。

首先我们建立一个类

这个类是ListViewDataTemplateSelector选择我们的模板，模板我们会在xaml，不会写cs

我们类继承DataTemplateSelector

我们属性

```

 public DataTemplate MaleData { set; get; }

 public DataTemplate FemaleData { set; get; }

```

然后我们判断我们是否传进来是男生，如果是就返回MaleData

我们需要override SelectTemplateCore，这时有两个，如果我们的 `ItemsPanel` 是 `ItemsStackPanel` 或 ItemsWrapGrid 我们就需要选择 `SelectTemplateCore(Object)` 。如果我们的 `VirtualizingStackPanel` 或其他的`WrapGrid` ，就是 `SelectTemplateCore(Object, DependencyObject)`

```
        protected override DataTemplate SelectTemplateCore(object item, DependencyObject container)
        {
            if (item is Male)
            {
                return MaleData;
            }
            return FemaleData;
        }
```

我们在xaml 

先写出两个 DataTemplate ，一定要有key，然后在ListViewDataTemplateSelector放在FemaleData，MaleData，注意也要key，随便给个名称都好

```
        <DataTemplate x:Key="MaleData">
            <Grid>
               <Border>
                   <Grid Margin="10,10,10,10">
                        <StackPanel>
                            <TextBlock Text="名称"></TextBlock>
                            <TextBlock Text="{Binding Path=Name}"></TextBlock>
                            <TextBlock Text="身高"></TextBlock>
                            <TextBlock Text="{Binding Path=Stature}"></TextBlock>
                        </StackPanel>
                    </Grid>
               </Border>
            </Grid>
        </DataTemplate>
        <DataTemplate x:Key="FemaleData">
            <Grid>
                <Border>
                    <Grid Margin="10,10,10,10">
                        <StackPanel>
                            <TextBlock Text="名称"></TextBlock>
                            <TextBlock Text="{Binding Path=Name}"></TextBlock>
                            <TextBlock Text="年龄"></TextBlock>
                            <TextBlock Text="{Binding Path=Year}"></TextBlock>
                        </StackPanel>
                    </Grid>
                </Border>
            </Grid>
        </DataTemplate>
        <local:ListViewDataTemplateSelector x:Key="Selector" FemaleData="{StaticResource FemaleData}"
                                            MaleData="{StaticResource MaleData}"></local:ListViewDataTemplateSelector>
```

我们必须给key，然后使用`{StaticResource }`在listView也是使用

```
        <ListView ItemsSource="{x:Bind View.HumanWorld}"
                  ItemTemplateSelector="{StaticResource Selector}"></ListView>
```


![这里写图片描述](http://img.blog.csdn.net/20160802195044703)

传进入的Item如果is Male就会使用MaleData，这样就是选择了我们的模板，可以在一个List使用不同的数据

源代码：https://github.com/lindexi/kechengbiao


<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://i.creativecommons.org/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。