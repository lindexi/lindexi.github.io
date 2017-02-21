# WPF 鼠标移动到列表上 显示列表图标

<!--more-->

在列表新建一个图标

`Visibility="{Binding RelativeSource={RelativeSource AncestorType=ListBoxItem}, Path=IsMouseOver, Converter={StaticResource BooleanToVisibilityConverter}}"`