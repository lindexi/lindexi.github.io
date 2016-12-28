# win10 uwp 修改Pivot Head 颜色

我们有十个Pivot，我们想修改所有的Pivot的颜色。那么我们可以在每个Pivot修改Template。

<!--more-->
<!-- csdn -->

这方法我就得不好。

假如我们有一个Pivot
		
```
        <Pivot>
            <Pivot.Items>
                <PivotItem Header="lindexi" ></PivotItem>
                <PivotItem Header="csdn" ></PivotItem>
            </Pivot.Items>
        </Pivot>

```
我们想修改颜色，可以使用
		
```
    <Pivot.HeaderTemplate>
        <DataTemplate>
            <TextBlock Text="{Binding}" Foreground="Cyan" FontSize="40" />
        </DataTemplate>
    </Pivot.HeaderTemplate>

```

![](http://7xqpl8.com1.z0.glb.clouddn.com/f3c2f4a3-94ae-40b4-b3c3-da73116eb75d2016121716265.jpg)

参见：
http://stackoverflow.com/questions/31797875/overriding-pivot-header-foreground-brushes-in-uwp-app-win-10-rtm-sdk

<!-- 
如果我们要修改很多个Pivot颜色

我们可以在App.xaml使用PivotHeaderForegroundSelectedBrush

		
```
    <SolidColorBrush x:Key="PivotHeaderForegroundSelectedBrush" Color="Red" />
    <SolidColorBrush x:Key="PivotHeaderForegroundUnselectedBrush" Color="Cyan" />
    <x:Double x:Key="PivotHeaderItemFontSize">40</x:Double>

```

我们新建一个PivotDictionary


![](http://7xqpl8.com1.z0.glb.clouddn.com/f3c2f4a3-94ae-40b4-b3c3-da73116eb75d20161217162759.jpg)

在里面写上面代码

然后在App.xaml使用

这样，所有的Pivot都使用我们上面的颜色 -->
