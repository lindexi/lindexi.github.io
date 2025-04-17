
# dotnet 写一个支持层层继承属性的对象

我最近在造一个比 Excel 差得多的表格控件，其中一个需求是属性的继承。大家都知道，表格里面有单元格，单元格里面允许放文本，文本可以放多段文本。本文的主角就是文本段落的样式属性，包括文本字体字号颜色等等属性。文本段落的属性，如果没有特别设置，将使用单元格里面的文本样式属性。而如果单元格里面，没有特别指定此单元格使用特殊的文本样式，将会继承使用当前所在的行的文本样式。如果当前行没有特殊指定文本样式属性，那么将会使用文档的默认样式。文档默认样式将会根据是否有特殊指定而采用主题样式
如此复杂的层层继承逻辑，如果每个属性都需要自己一层层去寻找，那代码量将会特别多。维护起来就想吃桌子

<!--more-->


<!-- CreateTime:2021/11/8 16:40:40 -->


<!-- 发布 -->

为了保住桌子，咱来写一个支持层层继承属性的对象。如在当前层找不到某个属性，将会往上一层自动去找，一层层找。如果都找不到，将返回默认值

以下是这个类的定义代码

```csharp
    public class FlattenObject
    {
        /// <summary>
        /// 创建带继承的对象
        /// </summary>
        /// <param name="reserved"></param>
        public FlattenObject(FlattenObject? reserved = null)
        {
            Reserved = reserved;
        }

        private FlattenObject? Reserved { get; }
        private Dictionary<string, object> ValueDictionary { get; } = new Dictionary<string, object>();

        /// <summary>
        /// 设置属性值
        /// </summary>
        /// <param name="value"></param>
        /// <param name="propertyName"></param>
        protected void SetValue(object value, [CallerMemberName] string propertyName = null!)
        {
            ValueDictionary[propertyName] = value;
        }

        /// <summary>
        /// 获取属性值
        /// </summary>
        /// <typeparam name="T"></typeparam>
        /// <param name="propertyName"></param>
        /// <returns></returns>
        protected T? GetValue<T>([CallerMemberName] string propertyName = null!)
            => GetValue<T>(default!, propertyName);

        /// <summary>
        /// 获取属性值
        /// </summary>
        /// <typeparam name="T"></typeparam>
        /// <param name="defaultValue"></param>
        /// <param name="propertyName"></param>
        /// <returns></returns>
        protected T GetValue<T>(T defaultValue, [CallerMemberName] string propertyName = null!)
        {
            if (ValueDictionary.TryGetValue(propertyName, out var value))
            {
                return (T) value;
            }
            else
            {
                if (Reserved is not null)
                {
                    return Reserved.GetValue<T>(defaultValue, propertyName);
                }
                else
                {
                    return defaultValue;
                }
            }
        }
    }
```

通过 Reserved 属性表示的是当前层的上一层的对象，用于给当前层进行继承。因为每一层都包含了上一层的对象，因此从最下层就可以一层层自动找到属性的值

继承当前类型，即可写出下面代码

```csharp
        class FooFlattenObject : FlattenObject
        {
            public FooFlattenObject(FlattenObject reserved = null) : base(reserved)
            {
            }

            public string FontName
            {
                set => SetValue(value);
                get => GetValue<string>();
            }

            public int Count
            {
                set => SetValue(value);
                get => GetValue<int>();
            }
        }
```

如上面代码，在各个属性的 set 和 get 都换成调用方法，而不需要定义字段

下面来尝试写单元测试

```csharp
            "给定可继承的对象，可以从继承的对象拿到属性值".Test(() =>
            {
                var reserved = new FooFlattenObject()
                {
                    FontName = "微软雅黑"
                };

                var fakeFlattenObject = new FooFlattenObject(reserved);
                Assert.AreEqual("微软雅黑", fakeFlattenObject.FontName);

                fakeFlattenObject.Count = 2;
                Assert.AreEqual(2, fakeFlattenObject.Count);
                Assert.AreEqual(0, reserved.Count);
            });
```

可以看到在 reserved 对象里面设置了 FontName 的值，可以被 fakeFlattenObject 继承拿到，同时自动读取的代码对于上层业务来说几乎没有

对 fakeFlattenObject 进行设置 Count 的值，不会影响到 reserved 对象

通过此方法可以让存在层层继承逻辑的代码不需要大量重复。除了在表格上使用，也可以用在如解析 PPT 的形状内文本，如 PPT 的图片裁剪等需要继承属性的逻辑上

上面的代码也存在不足，那就是对于结构体不友好，如 bool 或 int 等类型，都需要转换为 object 存放





<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。