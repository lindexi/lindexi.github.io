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