# C＃ 枚举转字符串

有时候需要把枚举转字符串，那么如何把枚举转字符串？
<div id="toc"></div>
<!--more-->
<!-- csdn -->

## 枚举转字符串

假如需要把枚举转字符串，可以直接把他进行转换，请看代码

```csharp
        public enum Di
        {
            /// <summary>
            /// 轨道
            /// </summary>
            Railway,

            /// <summary>
            /// 河流
            /// </summary>
            River,
        }

        static void Main(string[] args)
        {
            Console.WriteLine(Di.Railway.ToString());
        }
```

这样就可以把枚举转字符串

除了这个方法，可以使用 C# 6.0 的关键字，请看代码

```csharp
            Console.WriteLine(nameof(Di.Railway));

```

## 字符串转枚举

如果把一个枚举转字符串，那么如何把字符串转枚举？可以使用 `Enum.Parse` 不过这个方法可以会抛异常，所以使用需要知道字符串是可以转

```csharp

        public enum Di
        {
            /// <summary>
            /// 轨道
            /// </summary>
            Railway,

            /// <summary>
            /// 河流
            /// </summary>
            River,
        }

             static void Main(string[] args)
        {
            string str = Di.Railway.ToString();
            Console.WriteLine(Enum.Parse(typeof(Di), str).ToString());
        }
```
