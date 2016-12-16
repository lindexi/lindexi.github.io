# win10 uwp 绑定静态属性

Jasoon 大神问，如何绑定静态属性。

我们经常有静态属性，那么我们如何绑定

<!--more-->

假如我们的ViewModel有一个静态属性

		
```
        public static string CVTE
        {
            set;
            get;
        } = "lindexi";

```

<!-- 如果我们ViewModel namespace Bproperty.ViewModel

那么需要在空间

		
```
xmlns:view="using:Bproperty.ViewModel"

```

然后使用 -->

		
```
public  string Property => ViewModel.ViewModel.CVTE;

```
		
```
      <TextBlock Text="{x:Bind Property,Mode=OneWay}"></TextBlock>


```


参见：http://stackoverflow.com/questions/15854708/how-can-i-bind-a-xaml-property-to-a-static-variable-in-another-class

http://stackoverflow.com/questions/34701255/how-to-bind-to-attached-property-in-uwp