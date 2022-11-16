# MAUI 已知问题 PathFigureCollectionConverter 非线程安全

在 MAUI 里，可以使用 PathFigureCollectionConverter 将 Path 字符串转换为 PathFigureCollection 对象，从而实现从 Path 字符串转换为路径几何。然而这个 PathFigureCollectionConverter 转换器非线程安全，即使创建多个实例对象，依然由于内部的静态字段导致非线程安全，本文将记录此问题的原理

<!--more-->
<!-- CreateTime:2022/11/14 20:22:49 -->

<!-- 发布 -->
<!-- 博客 -->

当前，此 PathFigureCollectionConverter 非线程安全问题，没有规避方法

我将此问题报告给官方，请看 [https://github.com/dotnet/maui/issues/11321](https://github.com/dotnet/maui/issues/11321)

复现步骤：

只需要让 PathFigureCollectionConverter 进入多线程转换 Path 字符串即可，转换过程，将随机抛出异常

最简代码如下

```csharp
        var pathFigureCollectionConverter = new PathFigureCollectionConverter();
        var pathString1 =
            "M320 96a64 64 0 0 0-64 64V896a64 64 0 0 0 64 64h384a64 64 0 0 0 64-64V160a64 64 0 0 0-64-64H320z m384 64V896H320V160h384zM128 256v576h64V256H128zM832 256v576h64V256h-64z";
        var pathString2 =
            "M230.08 738.56l58.816 58.88L574.336 512l-285.44-285.44-58.88 58.88L456.704 512l-226.56 226.56z m487.04 29.44V256h-83.2v512h83.2z";

        _ = pathFigureCollectionConverter.ConvertFrom(pathString1);
        _ = pathFigureCollectionConverter.ConvertFrom(pathString2);

        for (int i = 0; i < 100; i++)
        {
            Task.Run(() =>
            {
                var pathFigureCollection = pathFigureCollectionConverter.ConvertFrom(pathString1);
            });
        }

        for (int i = 0; i < 100; i++)
        {
            Task.Run(() =>
            {
                var pathFigureCollection = pathFigureCollectionConverter.ConvertFrom(pathString2);
            });
        }
```

包含此代码的项目放在 GitHub 上，请看 [https://github.com/lindexi/lindexi_gd/tree/abc3042ddbfc3bd46563119fc88df0463b155c8b/TestPathFigureCollectionConverter](https://github.com/lindexi/lindexi_gd/tree/abc3042ddbfc3bd46563119fc88df0463b155c8b/TestPathFigureCollectionConverter)

导致 PathFigureCollectionConverter 非线程安全的核心原因是采用静态字段记录状态，以下是 PathFigureCollectionConverter 的部分源代码，代码有删减

```csharp
        static bool _figureStarted;
        static string _pathString;
        static int _pathLength;
        static int _curIndex;
        static Point _lastStart;
        static Point _lastPoint;
        static Point _secondLastPoint;
        static char _token;

        static void ParseToPathFigureCollection(PathFigureCollection pathFigureCollection, string pathString, int startIndex)
        {
            PathFigure pathFigure = null;

            _pathString = pathString;
            _pathLength = pathString.Length;
            _curIndex = startIndex;

            // 忽略其他代码
        }
```

完全的代码请看 [https://github.com/dotnet/maui/blob/a541df0816d1867f494186a0bdc214d431e000cd/src/Controls/src/Core/Shapes/PathFigureCollectionConverter.cs](https://github.com/dotnet/maui/blob/a541df0816d1867f494186a0bdc214d431e000cd/src/Controls/src/Core/Shapes/PathFigureCollectionConverter.cs)

从上面代码可以看到，用到了静态字段。这是非多线程安全的，多个线程将会随机更改污染静态字段，从而让转换逻辑无法成功执行

