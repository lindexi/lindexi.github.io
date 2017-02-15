# C＃枚举中使用Flags特性

如果对一个值可以包含多个，那么可以使用枚举，加上Flags

本文告诉大家如何写一个 Flags。


<!--more-->
<!-- csdn -->

在写前，需要知道一些基础知识，取反、或、与，如果不知道的话，请去看看基础。

当然，这些太复杂了，我也不会在这里解释。

假如有类型


```csharp
    [Flags]
    public enum Show
    {
        A = 0x00000001,
        B = 0x00000010,
        C = 0x00000100,
        D = 0x00001000,
    }
```


## 合并多个值

合并多个，使用 `|`


```csharp
  Show show=Show.A | Show.B
```



## 判断是否存在某个值

一个简单方法是用 HasFlag，但是一个方法是用 `&` 


```csharp
  Show show=Show.A | Show.B;
  show.HasFlag(Show.A);
  //其他
  bool 包含=(show & Show.A)!=0;
```


## 去掉一个值

```csharp
  Show show=Show.A | Show.B;
  show=show & (~Show.A);
```

## 取反一个值


```csharp
  Show show=Show.A | Show.B;
  bool 包含=(show & Show.A)!=0;
  if(包含)
  {
     show=show & (~Show.A);
  }
  else
  {
     show=show | Show.A;
  }
```


参见：http://www.cnblogs.com/jhxk/articles/1738831.html