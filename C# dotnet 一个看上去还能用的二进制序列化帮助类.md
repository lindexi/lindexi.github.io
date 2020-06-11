# C# dotnet 一个看上去还能用的二进制序列化帮助类

这仅是一个辅助方法帮助类，可以协助小伙伴写二进制序列化d额效率，代码也还看的过去

<!--more-->
<!-- 发布 -->

在开始之前，我需要说明的是，如果不是必要，不要使用二进制序列化。因为很难做到版本兼容，如果写错了也不知道是哪里写错了，调试难度很大。但是对于性能的提升，其实也不大

```csharp
    /// <summary>
    /// 二进制序列化
    /// </summary>
    interface IBinarySerializable
    {
        void Serialize(BinaryWriter binaryWriter);
        void Deserialize(BinaryReader binaryReader);
    }
```

这个接口用于给对象继承，如果对象继承了，那么就方便进行序列化

这是一些辅助方法

没有用到反射，需要自己手动写转换代码。注意顺序

```csharp
    static class BinarySerialize
    {
        /// <summary>
        /// 写入 uin32 列表
        /// </summary>
        /// <param name="binaryWriter"></param>
        /// <param name="list"></param>
        public static void WriteUint32List(this BinaryWriter binaryWriter, List<uint> list)
        {
            // 格式先写入列表长度，然后依次写入内容
            binaryWriter.Write(list.Count);

            foreach (var n in list)
            {
                binaryWriter.Write(n);
            }
        }

        /// <summary>
        /// 读取 uint32 列表
        /// </summary>
        /// <param name="binaryReader"></param>
        /// <returns></returns>
        public static List<uint> ReadUint32List(this BinaryReader binaryReader)
        {
            // 读取长度
            var count = binaryReader.ReadInt32();

            List<uint> list = new List<uint>(count);
            for (int i = 0; i < count; i++)
            {
                list.Add(binaryReader.ReadUInt32());
            }

            return list;
        }

        /// <summary>
        /// 写入字符串列表
        /// </summary>
        /// <param name="binaryWriter"></param>
        /// <param name="stringList"></param>
        public static void WriteStringList(this BinaryWriter binaryWriter, List<string> stringList)
        {
            // 格式先写入列表长度，然后依次写入字符串
            binaryWriter.Write(stringList.Count);

            foreach (var str in stringList)
            {
                binaryWriter.Write(str);
            }
        }

        /// <summary>
        /// 写入可序列化类的列表
        /// </summary>
        /// <typeparam name="T"></typeparam>
        /// <param name="binaryWriter"></param>
        /// <param name="list"></param>
        public static void WriteList<T>(this BinaryWriter binaryWriter, List<T> list) where T : IBinarySerializable
        {
            // 先写入长度
            binaryWriter.Write(list.Count);
            foreach (var binarySerializable in list)
            {
                binarySerializable.Serialize(binaryWriter);
            }
        }

        /// <summary>
        /// 读取可序列化类的列表
        /// </summary>
        /// <typeparam name="T"></typeparam>
        /// <param name="binaryReader"></param>
        /// <returns></returns>
        public static List<T> ReadList<T>(this BinaryReader binaryReader) where T : IBinarySerializable, new()
        {
            // 读取长度
            var count = binaryReader.ReadInt32();
            List<T> list = new List<T>(count);
            for (int i = 0; i < count; i++)
            {
                T t = new T();
                t.Deserialize(binaryReader);
                list.Add(t);
            }

            return list;
        }

        /// <summary>
        /// 读取字符串列表
        /// </summary>
        /// <param name="binaryReader"></param>
        /// <returns></returns>
        public static List<string> ReadStringList(this BinaryReader binaryReader)
        {
            // 先读取列表长度，然后依次读取字符串
            var count = binaryReader.ReadInt32();
            List<string> stringList = new List<string>(count);
            for (int i = 0; i < count; i++)
            {
                var str = binaryReader.ReadString();
                stringList.Add(str);
            }

            return stringList;
        }

        /// <summary>
        /// 写入头信息，默认是需要固定长度的
        /// </summary>
        /// <param name="binaryWriter"></param>
        /// <param name="head"></param>
        /// <param name="headLength"></param>
        public static void WriteHead(this BinaryWriter binaryWriter, string head, int headLength)
        {
            var headByte = StringToByteList(head);
            binaryWriter.WriteByteList(headByte, headLength);
        }

        /// <summary>
        /// 将字符串转换为 byte 其中字符串添加长度前缀
        /// </summary>
        /// <param name="str"></param>
        /// <returns></returns>
        public static byte[] StringToByteList(string str)
        {
            var strByteList = Utf8.GetBytes(str);
            ushort byteLength = (ushort)strByteList.Length;
            var newLength = sizeof(ushort) + strByteList.Length;

            var byteList = new byte[newLength];
            var binaryWriter = new BinaryWriter(new MemoryStream(byteList));
            binaryWriter.Write(byteLength);
            binaryWriter.Write(strByteList);
            return byteList;
        }

        /// <summary>
        /// 写入二进制写入固定长度
        /// </summary>
        /// <param name="binaryWriter"></param>
        /// <param name="source"></param>
        /// <param name="byteCount"></param>
        public static void WriteByteList(this BinaryWriter binaryWriter, byte[] source, int byteCount)
        {
            var byteList = new byte[byteCount];
            Array.Copy(source, 0, byteList, 0, Math.Min(source.Length, byteCount));

            binaryWriter.Write(byteList, 0, byteCount);
        }

        private static Encoding Utf8 => Encoding.UTF8;

        /// <summary>
        /// 读取固定长度的头
        /// </summary>
        /// <param name="binaryReader"></param>
        /// <param name="headLength"></param>
        /// <returns></returns>
        public static string ReadHead(this BinaryReader binaryReader, int headLength)
        {
            var strByteLength = (ushort)binaryReader.ReadInt16();
            var strByteList = binaryReader.ReadBytes(strByteLength);
            var head = Utf8.GetString(strByteList);

            var readLength = headLength - sizeof(ushort) - strByteLength;
            binaryReader.ReadBytes(readLength);

            return head;
        }
    }
```

这里的读写 Head 也许小伙伴的业务是用不到的，我用这个方法主要是写入版本号

这是单元测试的代码，只是测试主要使用方法，边界没有测试

```csharp
    [TestClass]
    public class BinarySerializeTests
    {
        private class FakeBinarySerialize : IBinarySerializable, IEquatable<FakeBinarySerialize>
        {
            /// <inheritdoc />
            public void Serialize(BinaryWriter binaryWriter)
            {
                binaryWriter.Write(F1);
                binaryWriter.Write(F2);
            }

            /// <inheritdoc />
            public void Deserialize(BinaryReader binaryReader)
            {
                F1 = binaryReader.ReadString();
                F2 = binaryReader.ReadInt32();
            }

            public string F1 { set; get; }
            public int F2 { set; get; }

            /// <inheritdoc />
            public bool Equals(FakeBinarySerialize other)
            {
                if (ReferenceEquals(null, other)) return false;
                if (ReferenceEquals(this, other)) return true;
                return F1 == other.F1 && F2 == other.F2;
            }

            /// <inheritdoc />
            public override bool Equals(object obj)
            {
                if (ReferenceEquals(null, obj)) return false;
                if (ReferenceEquals(this, obj)) return true;
                if (obj.GetType() != this.GetType()) return false;
                return Equals((FakeBinarySerialize)obj);
            }

            /// <inheritdoc />
            public override int GetHashCode()
            {
                unchecked
                {
                    return ((F1 != null ? F1.GetHashCode() : 0) * 397) ^ F2;
                }
            }
        }

        [ContractTestCase]
        public void WriteUint32List()
        {
            "写入uint列表，可以读取列表内容".Test(() =>
            {
                // Arrange
                var memoryStream = new MemoryStream();
                var binaryWriter = new BinaryWriter(memoryStream);
                var fakeBinarySerializeList = new List<uint>();
                for (int i = 0; i < 100; i++)
                {
                    fakeBinarySerializeList.Add((uint)i);
                }

                // Action
                binaryWriter.WriteUint32List(fakeBinarySerializeList);

                // Assert
                memoryStream.Seek(0, SeekOrigin.Begin);
                var binaryReader = new BinaryReader(memoryStream);
                var readList = binaryReader.ReadUint32List();

                Equal(fakeBinarySerializeList, readList);
            });
        }

        [ContractTestCase]
        public void WriteList()
        {
            "写入列表内容，可以读取列表".Test(() =>
            {
                // Arrange
                var memoryStream = new MemoryStream();
                var binaryWriter = new BinaryWriter(memoryStream);
                var fakeBinarySerializeList = new List<FakeBinarySerialize>();
                for (int i = 0; i < 100; i++)
                {
                    fakeBinarySerializeList.Add(new FakeBinarySerialize()
                    {
                        F1 = i.ToString(),
                        F2 = i,
                    });
                }

                // Action
                binaryWriter.WriteList(fakeBinarySerializeList);

                // Assert
                memoryStream.Seek(0, SeekOrigin.Begin);
                var binaryReader = new BinaryReader(memoryStream);
                var readList = binaryReader.ReadList<FakeBinarySerialize>();
                Equal(fakeBinarySerializeList, readList);
            });
        }

        private void Equal<T>(List<T> a, List<T> b)
        {
            Assert.AreEqual(a.Count, b.Count);
            for (int i = 0; i < a.Count; i++)
            {
                Assert.AreEqual(a[i], b[i]);
            }
        }

        [ContractTestCase]
        public void WriteStringList()
        {
            "写入空字符串列表，可以读取空的列表".Test(() =>
            {
                // Arrange
                var memoryStream = new MemoryStream();
                var binaryWriter = new BinaryWriter(memoryStream);
                var head = "Font Data 1.0.0";
                var length = 20;
                var test = (byte)0xF1;

                var stringList = new List<string>();

                // Action
                binaryWriter.WriteHead(head, length);
                binaryWriter.Write(test);
                binaryWriter.WriteStringList(stringList);

                // Assert

                memoryStream.Seek(0, SeekOrigin.Begin);
                var binaryReader = new BinaryReader(memoryStream);
                var str = binaryReader.ReadHead(length);
                var b = binaryReader.ReadByte();
                var readList = binaryReader.ReadStringList();

                Assert.AreEqual(0, readList.Count);
            });

            "写入字符串列表，可以读取写入的值".Test(() =>
            {
                // Arrange
                var memoryStream = new MemoryStream();
                var binaryWriter = new BinaryWriter(memoryStream);
                var head = "Font Data 1.0.0";
                var length = 20;
                var test = (byte)0xF1;

                var stringList = new List<string>()
                {
                    "lindexi","doubi"
                };

                // Action
                binaryWriter.WriteHead(head, length);
                binaryWriter.Write(test);
                binaryWriter.WriteStringList(stringList);

                // Assert
                memoryStream.Seek(0, SeekOrigin.Begin);
                var binaryReader = new BinaryReader(memoryStream);
                var str = binaryReader.ReadHead(length);
                var b = binaryReader.ReadByte();
                var readList = binaryReader.ReadStringList();
            });
        }

        [ContractTestCase]
        public void WriteHead()
        {
            "尝试写入长度小于指定长度的头，可以写入和读取".Test(() =>
            {
                // Arrange
                var memoryStream = new MemoryStream();
                var binaryWriter = new BinaryWriter(memoryStream);
                var head = "Font Data 1.0.0";
                var length = 20;
                var test = (byte)0xF1;
                // Action
                binaryWriter.WriteHead(head, length);
                binaryWriter.Write(test);

                // Assert
                memoryStream.Seek(0, SeekOrigin.Begin);
                var binaryReader = new BinaryReader(memoryStream);
                var str = binaryReader.ReadHead(length);
                var b = binaryReader.ReadByte();

                Assert.AreEqual(head, str);
                Assert.AreEqual(test, b);
            });
        }
    }
```

上面代码需要使用 [CUnit](github.com/dotnet-campus/CUnit) 库的支持，才能在单元测试里面使用中文写条件

```xml
        <PackageReference Include="MSTestEnhancer" Version="1.6.0" />
```

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
