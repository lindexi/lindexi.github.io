我朋友在做一个控件，是显示异常，那么异常多就变为颜色，大概就是下面的图，很简单

## xaml定义常量

我们如何在我们界面定义一个常量，我有很多地方需要用到一个常量，那么我如何定义一个，让修改只有一个，不需要整个界面都在修改。

在WPF我们使用常量可以使用

```
<Page
 xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
 xmlns:sys="clr-namespace:System;assembly=mscorlib"  
 xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml">
   <Page.Resources>
     <sys:Double x:Key="Height">200</sys:Double> 
     <sys:Double x:Key="Width">200</sys:Double>
   </Page.Resources> 
 <Grid> 
  <Rectangle Height="{StaticResource Height}" Width="{StaticResource Width}" Fill="Blue"/> 
 </Grid>
</Page>

```

在UWP那简单，我们在Resource

```
 <x:Double x:Key="Height"> 200 </x:Double>

```

当然需要一个Key，然后一个值，我们可以有

- Boolean

- Int32

- String

