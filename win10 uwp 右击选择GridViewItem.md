# win10 uwp 右击选择GridViewItem

有时候我们需要选择一个GridView的一项，通过我们右击。


<!--more-->

于是我们需要在GridView的SelectionMode为Single，IsRightTapEnabled为True

假如我们给的ItemsSource的类型是`List<Student>`，那我们可以通过简单方法得到右击的Student。

我们需要使用RightTapped

		
```
      <GridView x:Name="SymbolGridView"
         SelectionMode="Single"
         IsItemClickEnabled="True"
         IsRightTapEnabled="True"
         ItemsSource="{x:Bind View.Student}"
         ItemClick="SymbolGridView_OnItemClick"
         RightTapped="SymbolGridView_OnRightTapped">
            <GridView.ItemTemplate>
                <DataTemplate x:DataType="view:ViewModel">
                    <TextBlock Text="{Binding Name}"></TextBlock>
                    </DataTemplate>
            </GridView.ItemTemplate>
        </GridView>

```

注意DataTemplate的是TextBlock

我们通过
		
```
        private void SymbolGridView_OnRightTapped(object sender, RightTappedRoutedEventArgs e)
        {
            var student = (e.OriginalSource as TextBlock)?.DataContext as Student;
        }

```

就可以得到Student

注意`e.OriginalSource`就是我们刚才使用的DateTemplate的TextBlock，我们在DateTemplate使用类型Type，那么OriginalSource就可以使用Type。拿到后，他的DataContext就是我们选择的。

如果使用个人控件（UserControl），那么请要有DataContext，不要覆盖。

这样我们就可以得到GridViewItem

但有时候，OriginalSource是ListViewItemPresenter，我们可以用一个简单方法，使用FrameworkElement

我们修改代码
        
```
 var student = (e.OriginalSource as FrameworkElement)?.DataContext as Student;

```

这样我们就可以得到，不需要去看DataTemplate

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://i.creativecommons.org/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。 