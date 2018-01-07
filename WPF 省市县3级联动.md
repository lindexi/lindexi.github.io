# WPF 省市县3级联动

本文告诉大家如何使用绑定做省市县3级联动，代码从网上找的。

<!--more-->
<!-- csdn -->

首先定义显示的类，包括 id 和 名称 

```csharp
    public class CodeView
    {
        public string Id { get; set; }

        public string Name { get; set; }
    }
```

然后定义省市县的数据

```csharp
    public class Province: CodeView
    {
        public List<City> Child { get; set; }
    }

    public class City: CodeView
    {
        public List<County> Child { get; set; }
    }

    public class County:CodeView
    {

    }
```

因为可以通过 xaml 绑定 选择的元素，所以可以绑定


数据：[省市区](http://7xqpl8.com1.z0.glb.clouddn.com/%E7%9C%81%E5%B8%82%E5%8C%BA.7z )

感谢 Baolaitong 提供代码