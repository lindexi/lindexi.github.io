# win10 uwp 绑定静态属性

Jasoon 大神问，如何绑定静态属性。

我们经常有静态属性，那么我们如何绑定

<!--more-->

<div id="toc"></div>
<!-- csdn -->

假如我们的ViewModel有一个静态属性

		

```csharp
        public static string CVTE
        {
            set;
            get;
        } = "lindexi";

```

<!-- 如果我们ViewModel namespace Bproperty.ViewModel

那么需要在空间

		

```csharp
xmlns:view="using:Bproperty.ViewModel"

```

然后使用 -->

一个方法是在xaml.cs写一个属性，get为静态属性。
		

```csharp
public  string Property => ViewModel.ViewModel.CVTE;

```
在xaml绑定xaml.cs属性

```xml
      <TextBlock Text="{x:Bind Property,Mode=OneWay}"></TextBlock>


```

下面方法我没测试成功，但是有大神说他这样做是可以。

在需要绑定的类上写`[Bindable]`

这样写的具体参见：http://www.jaylee.org/post/2012/03/07/Xaml-integration-with-WinRT-and-the-IXamlMetadataProvider-interface.aspx 

我不知道为何这样，知道的大神可以告诉我不？

然后在xaml写

        

```csharp
xmlns:view="using:Bproperty.ViewModel"

 <TextBlock Text="{x:Bind Path=(view:ViewModel.CVTE) }"
               HorizontalAlignment="Center" VerticalAlignment="Center"/>

```

参见：http://stackoverflow.com/questions/15854708/how-can-i-bind-a-xaml-property-to-a-static-variable-in-another-class

http://stackoverflow.com/questions/34701255/how-to-bind-to-attached-property-in-uwp

