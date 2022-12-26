# dotnet 世界猜测 随机数的小测试

这是一个半技术向的博客，主题来源于我读过的某本书的片段，这是一个稍稍有些前置知识的故事，主题的大概内容就是假定世界存在某个规则序列，通过一代代的探索，可以获取到此序列的内容。本文将模拟此情形，写一个随机数的小测试应用，来不严谨的证明，或者准确来说是用代码讲述这个故事

<!--more-->
<!-- CreateTime:2020/2/29 10:27:27 -->

<!-- 发布 -->
<!-- 博客 -->

天有五贼，见之者昌。 传说，在一个由某逗写的代码世界里面，世界里藏着一段取值范围为 0 到 100 且长度为 10 的序列。某逗将此序列称为世界序列。在此世界里面，还生活着一群名为 Element 的小人儿。这些 Element 小人儿，都在探索和追求着世界序列。因为呀，每当此世界经过一代的时候，嗯，约等于某个大循环执行一遍的时候，将会根据此世界序列的内容，再加上某逗规定的算法，决定哪些 Element 小人儿需要被淘汰掉。已知的是，如果 Element 小人儿越了解准确世界序列，那 Element 小人儿就越能生存下来

此世界每经过一代，将会淘汰一批 Element 小人儿。而存活下来的 Element 小人儿将会迎来新的一轮生娃过程。可以由存活下来的 Element 小人儿创建出更多的下一代的 Element 小人儿

在经过此世界的很多代之后，咱来看看存活下来的 Element 小人儿是否都掌握了正确的世界序列

接下来就是写代码的内容了，命题确定了，其实通过简单的大学数学的知识，是能够计算出结果的，只不过咱是计算机系的，肯定要写点代码啦

按照本文的题注，天有五贼，见之者昌，先用代码写天有五贼。我的文学修养还不够，不能达意的解释天有五贼，见之者昌这句话。只能用代码的方式，以一个片面的方式来解释。在某逗写的代码世界里面，此世界的世界序列就是对应着天有五贼的意思。这片天地有五贼，五贼大概就是金木水火土五行的意思，也等于说世界的规则或者说世界的本源。本文的世界序列，就大概属于某逗写的代码世界里面的本源。而见之者昌可以认为是，如果有人可以知晓五贼，约等于说如果有人可以知道世界的规则或者说世界的本源，那他将会昌盛。对应某逗写的代码世界里面的 Element 小人儿，如果 Element 小人儿能够知晓世界序列，那他更能在此代码世界的一代代循环里面存活，而且 Element 小人儿也就能生产更多的下一代的 Element 小人儿，不断昌盛

某逗写的代码世界里的世界序列，序列里面的每个项，都可以用一个 Key 的只读结构体来表示。之所以不直接使用 int 等来表示，那是为了给一个单位，这属于程序员的修养。如以下代码定义了 Key 这个只读结构体，本文代码基于 dotnet 7 版本编写，本文代码都在 GitHub 上开源

```csharp
readonly record struct Key(int N)
{
    public const int MaxKeyValue = 100;
}
```

世界序列也就是一个 `List<Key>` 类型。当然，有了代码世界，为了代码世界能够好好工作，还需要一个世界管理者，也就是以下代码定义的 Manager 类型，在 Manager 里面就存放了世界序列，代码如下

```csharp
class Manager
{
    public List<Key> KeyList { get; }
    ... // 忽略其他代码
}
```

生活在代码世界里面的是一群 Element 小人儿，使用的是 Element 类型，每个 Element 小人儿都有自己的一段序列。这就是 Element 小人儿最重要的，也是不断探索的内容

```csharp
class Element
{
    public List<Key> KeyList { get; } = new List<Key>();
    ... // 忽略其他代码
}
```

在代码世界刚开始的时候，也就是 Manager 被创建出来的时候，将会立刻初始化世界序列

```csharp
    public Manager()
    {
        KeyList = new List<Key>();
        for (int i = 0; i < 10; i++)
        {
            var key = Random.Shared.Next(Key.MaxKeyValue);
            KeyList.Add(new Key(key));
        }
    }
```

世界开始的时候，将会进入 Manager 的 Start 方法，在初始化世界时，将会创建出首批的 Element 小人儿，如以下代码，由于怕电脑被玩坏，这里约束了 Element 小人儿的数量大概只有 10000 个

```csharp
class Manager
{
    public void Start()
    {
        for (int i = 0; i < 10000; i++)
        {
            var element = CreateElement();
            ElementList.Add(element);
        }
    }

    public List<Element> ElementList { get; } = new List<Element>();

    ... // 忽略其他代码
}
```

以上的 CreateElement 方法是一个用来辅助创建 Element 小人儿对象的方法，在实现他之前，还请先看看 Element 类型的构造函数吧

在 Element 类型里面，不仅包含了 KeyList 属性，还包括了两个辅助代码运行的属性，分别是 Random 随机数属性和 FinishBuildKey 属性，还请让我卖个关子，在本文后续部分再来揭晓 FinishBuildKey 属性的作用。在 Element 类型的构造函数里面，将要求传入 Random 属性，以让 Element 在每一代里面，拥有猜测世界序列的能力，且多个 Element 之间有奇妙的关系

```csharp
class Element
{
    public Element(Random random)
    {
        Random = random;
    }

    public Random Random { get; }

    private bool FinishBuildKey { get; set; }

    public List<Key> KeyList { get; } = new List<Key>();

    ... // 忽略其他代码
}
```

在了解到 Element 的构造函数之后，相信大家也能猜到 CreateElement 方法的实现了

```csharp
    private Element CreateElement()
    {
        Element element = new Element(new Random(Random.Shared.Next()));
        return element;
    }
```

在完成了代码世界的首批 Element 小人儿创建之后，就要进入激烈的世界迭代了。先创建一个大循环，大概是 10000 次。表示这个世界将会开始迭代 10000 次

```csharp
        for (int i = 0; i < 10000; i++)
        {
            ... // 忽略其他代码
        }
```

在每一代的开始时，都会让每个 Element 小人儿进行一轮思考，让 Element 决定是否在自己的 KeyList 里面，加上新的 Key 值。当然，每个 Element 小人儿的思考方式，现在就是通过 Random 随机数生成的哈

```csharp
        for (int i = 0; i < 10000; i++)
        {
            foreach (var element in ElementList)
            {
                element.BuildKey();
            }
            ... // 忽略其他代码
        }
```

以上的 BuildKey 函数就是放在 Element 里的函数，用于让 Element 小人儿思考新的 Key 值是什么，大概的实现如下

```csharp
class Element
{
    public bool BuildKey()
    {
        var key = Random.Next(Key.MaxKeyValue);
        KeyList.Add(new Key(key));

        return true;
    }
    ... // 忽略其他代码
}
```

当然了，有些 Element 小人儿决定放弃思考，也就是 Element 可以不再生成新的 Key 值。这也就是上文卖关子的 FinishBuildKey 属性的作用，此属性用来判断是否此 Element 已不再生成新的 Key 值了，约等于此 Element 认为自己已猜出了世界序列

```csharp
class Element
{
    public bool BuildKey()
    {
        if (KeyList.Count > 0 && (FinishBuildKey || Random.Shared.Next(10) == 1))
        {
            FinishBuildKey = true;
            return false;
        }

        var key = Random.Next(Key.MaxKeyValue);
        KeyList.Add(new Key(key));

        return true;
    }
    ... // 忽略其他代码
}
```

等待各个 Element 思考完成之后，此代码世界的考验就要开始了。在此世界的天道规则是从当前的世界序列里面通过当前世界的代数，也就是世界大循环的 i 变量的值，决定出世界序列中的一个 Key 值。对于每个 Element 来说，也要根据当前世界的代数返回一个 Key 值。一旦 Element 的返回的 Key 和此世界的 Key 值不相同，那么此 Element 将会被淘汰

说起来简单，先来看看代码如何实现

先实现从当前的世界序列里面通过当前世界的代数，也就是世界大循环的 i 变量的值，决定出世界序列中的一个 Key 值

```csharp
class Manager
{
    public void Start()
    {
        ... // 忽略其他代码
        for (int i = 0; i < 10000; i++)
        {
            ... // 忽略其他代码
            var key = GetKey(i);
            ... // 忽略其他代码
        }
    }

    public Key GetKey(int n)
    {
        var index = n % KeyList.Count;
        return KeyList[index];
    }

    ... // 忽略其他代码
}
```

每个 Element 也实现 GetKey 方法，也是通过传入的序号返回一个 Key 值

```csharp
class Element
{
    public Key GetKey(int n)
    {
        var index = n % KeyList.Count;
        return KeyList[index];
    }
    ... // 忽略其他代码
}
```

判断 Element 返回的 Key 是否和世界的相同，如果不相同，将会淘汰。代码只需要一句话就可以完成实现，那就是返回 Key 不相同的，从 Manager 的 ElementList 列表里面移除

```csharp
class Manager
{
    public void Start()
    {
        ... // 忽略其他代码
        for (int i = 0; i < 10000; i++)
        {
            ... // 忽略其他代码
            var key = GetKey(i);
            ElementList.RemoveAll(element => element.GetKey(i).N != key.N);
            ... // 忽略其他代码
        }
    }

    ... // 忽略其他代码
}
```

对于从 ElementList 被移除的 Element 来说，等待这些 Element 就是凌驾于此世界之上的 dotnet 的 GC 回收机制将其当初垃圾清理掉

经过了世界的一次考验，最后剩下的 Element 不多。此世界显得十分空旷，就给了这些存活的 Element 创建子 Element 的机会，在 Element 里实现 Create 方法，用来通过当前的 Element 创建新的 Element 对象

```csharp
class Element
{
    public Element Create()
    {
        Element element = new Element(Random);
        foreach (var key in KeyList)
        {
            element.KeyList.Add(key);
        }
        return element;
    }
    ... // 忽略其他代码
}
```

通过以上代码可以看到，新创建的 Element 不仅继承了用来思考的 Random 随机数，还继承了前辈对世界序列的思考，也就是 KeyList 元素。接下来新的 Element 可以继续在前辈的基础上，对世界序列进行思考

在 Manager 完成考验，将调用 Element 的 Create 方法，让此世界继续被填充

```csharp
class Manager
{
    public void Start()
    {
        ... // 忽略其他代码
        for (int i = 0; i < 10000; i++)
        {
            ... // 忽略其他代码
            var currentCount = ElementList.Count;
            while (ElementList.Count < 10000)
            {
                for (int index = 0; index < currentCount; index++)
                {
                    var element = ElementList[index];
                    ElementList.Add(element.Create());
                }
            }
            ... // 忽略其他代码
        }
    }

    ... // 忽略其他代码
}
```

如果当前的世界考验，干掉了绝大部分的 Element 对象，那么剩下的 Element 就有更多的机会创建出新的 Element 对象

为了防止灭世，也就是某次考验干掉了此世界上所有 Element 小人儿，在每次迭代的时候，如果世界太过空旷，将会额外加上从空白里创建的新 Element 对象

和世界的首批 Element 一样从石头蹦出来的，每次迭代，都会尝试加上一些从石头里面蹦出来的新 Element 对象，更改后的代码如下

```csharp
class Manager
{
    public void Start()
    {
        ... // 忽略其他代码
        for (int i = 0; i < 10000; i++)
        {
            ... // 忽略其他代码
            bool addElement = false;
            var currentCount = ElementList.Count;
            while (ElementList.Count < 10000)
            {
                for (int index = 0; index < currentCount; index++)
                {
                    var element = ElementList[index];
                    ElementList.Add(element.Create());
                }

                if (addElement)
                {
                    continue;
                }

                addElement = true;
                var addCount = 10000 - ElementList.Count;
                addCount = Math.Min(addCount, 100);
                for (int index = 0; index < addCount; index++)
                {
                    var element = CreateElement();
                    ElementList.Add(element);
                }
            }
            ... // 忽略其他代码
        }
    }

    ... // 忽略其他代码
}
```

如此的进行迭代，在完成了世界的大循环，也就是 10000 次之后，将会开始最后的审判。判断剩下的 Element 是否了解了世界序列，以及了解了多少次

```csharp
class Manager
{
    public void Start()
    {
        ... // 忽略其他代码
        for (int i = 0; i < 10000; i++)
        {
            ... // 忽略其他代码
        }

        for (var i = 0; i < KeyList.Count; i++)
        {
            var key = GetKey(i);
            ElementList.RemoveAll(element => element.GetKey(i).N != key.N);
        }
    }

    ... // 忽略其他代码
}
```

最后剩下的，都是明了世界序列，也就是明了此世界本质的 Element 小人儿

完成代码编写之后，放入 Main 函数用来启动世界

```csharp
internal class Program
{
    static void Main(string[] args)
    {
        Manager manager = new Manager();
        manager.Start();
    }
}
```

此代码版本，放在 GitHub 上，可以从 [https://github.com/lindexi/lindexi_gd/commit/49878e97df5c75c22d40294b6970aaf46b11c218](https://github.com/lindexi/lindexi_gd/commit/49878e97df5c75c22d40294b6970aaf46b11c218) 获取全部代码

运行代码的结果是，最后剩下的 Element 小人儿，都明了世界序列。其实这也是因为继承了前辈们的知识，从前辈那里拿到了 KeyList 才让越后创建的 Element 有越高的生存率

以上是一个简单的版本，世界序列是非常裸的参与计算，或者说没有参与计算，就根据序列进行返回。接下来对这个世界的游戏规则加上更多的难度，应用上加法规则。加法规则就是取随机的数值，例如 3 个数值，作为序号，再根据序号一一取出 Key 值，接着将 Key 值取和，返回一个数值。此规则同时也对每个 Element 执行，一旦发现 Element 计算出来的最终数值和世界计算出来的不匹配，那就将此 Element 淘汰

新的这个游戏规则其实对 Element 来说，更有挑战性，也同时带来了新的数学上的计算方法，那就是如果 Element 猜测的世界序列和此世界的世界序列不匹配，也有可能在取出的数值里面，通过加法返回相同的值。换句话说，每个 Element 在一轮迭代里面，如果没有被淘汰，那也是无法知道当前猜测的世界序列是否正确。对比之前的规则，之前的规则，一旦猜测错误，自然就会被淘汰

改造一下代码，让 Element 和 Manager 都继承 IKeyManager 接口，方便同时应用上相同的加法规则。这个 IKeyManager 接口定义如下

```csharp
interface IKeyManager
{
    Key GetKey(int n);
}
```

加法计算规则的代码实现如下，根据传入参数的序列，获取的 Key 值，取 Key 值的总和

```csharp
class Manager
{
    private int BuildByKey(IKeyManager keyManager, IList<int> indexList)
    {
        var n = 0;
        foreach (var index in indexList)
        {
            var key = keyManager.GetKey(index);
            n += key.N;
        }

        return n;
    }

    ... // 忽略其他代码
}
```

看到这里，也许机智的大家也就猜到了。世界序列对应着世界的本源数据，而加法规则对应的通过世界本源数据进行的一套规则。有世界本源数据加上世界的本源规则，即可展现出有趣的世界。有世界的本源规则的存在，也就是此代码世界的加法规则的存在，将会让 Element 小人儿更加难以知道自己所了解的世界规则是对是错

按照以上的加法规则，只要几个 Key 相加的和相等即可，而从数学上，这是无法反算唯一解的。举个例子，假定 indexList 里面有三个序号，分别是 1 2 3 三个序号。而 keyManager 里面根据 1 2 3 返回的 Key 分别是 10 20 30 三个 Key 值，计算出的结果是 60 的值。那反过来，已知返回值是 60 且传入序号是 1 2 3 三个序号。请问 keyManager 存放的 KeyList 里面的第 1 和 第 2 和 第 3 的 Key 应该是多少？ 抽象出来的数学题就是，已知三个数加起来的总和是 60 求这三个数。读过小学数学的大家，自然就知道，这三个数没有唯一解

如此加法规则，一开始就要求取序列里面的几个 Key 值，这就要求 Element 小人儿需要一开始就初始化一段 Key 列表。否则就不好玩了。经过实际的测试结果，我发现如果不告诉 Element 小人儿 世界序列的长度 的话，那 Element 小人儿 几乎不能在世界大循环结束之前，明了世界序列。降低游戏的难度，我将告诉 Element 小人儿 世界序列 的长度是 10 个 Key 值。同时也更改了 Element 的 BuildKey 方法，让此方法可以在一开始就生成了序列

在之前的代码基础上，修改 Element 的 BuildKey 方法，本来是 BuildKey 方法每次世界迭代都会调用，现在修改为只有 Element 被创建时才调用，修改过之后的代码如下，以下的代码还不是最终的版本，在下文将会告诉大家最终的版本代码

```csharp
class Element
{
    public void BuildKey()
    {
        const int count = 10;
        while (KeyList.Count < count)
        {
            var key = Random.Next(Key.MaxKeyValue);
            KeyList.Add(new Key(key));
        }
    }
    ... // 忽略其他代码
}
```

以上的代码可以让 Element 创建出来一段序列，序列长度和世界序列长度相同

接着从世界循环里面，删掉每次迭代都调用 BuildKey 方法的代码，也就是这段代码

```csharp
            foreach (var element in ElementList)
            {
                element.BuildKey();
            }
```

将调用 BuildKey 方法的代码放入到 CreateElement 方法里，修改之后的方法代码如下

```csharp
class Manager
{
    private Element CreateElement()
    {
        Element element = new Element(new Random(Random.Shared.Next()));
        element.BuildKey();
        return element;
    }
    ... // 忽略其他代码
}
```

每次世界迭代都会取出随机三个数值，将这三个随机数值传入到 BuildByKey 方法里，通过加法规则算出总和。将此方式分别应用在世界序列和每个 Element 上，将计算结果和世界序列计算出来的不相同的 Element 淘汰掉

实现的代码如下，每次迭代随机三个数值，可以先放入到一个数组里面

```csharp
class Manager
{
    public void Start()
    {
        ... // 忽略其他代码
        var indexList = new int[3];
        for (int i = 0; i < 10000; i++)
        {
            ... // 忽略其他代码
        }
    }

    ... // 忽略其他代码
}
```

每次调用 UpdateRandomIndexList 方法，将 indexList 加上三个随机数值

```csharp
class Manager
{
    public void Start()
    {
        ... // 忽略其他代码
        var indexList = new int[3];
        for (int i = 0; i < 10000; i++)
        {
            UpdateRandomIndexList(indexList);
            ... // 忽略其他代码
        }
    }

    private void UpdateRandomIndexList(int[] indexList)
    {
        for (var i = 0; i < indexList.Length; i++)
        {
            indexList[i] = Random.Shared.Next();
        }
    }

    ... // 忽略其他代码
}
```

先计算世界序列经过 BuildByKey 的值

```csharp
class Manager
{
    public void Start()
    {
        ... // 忽略其他代码
        var indexList = new int[3];
        for (int i = 0; i < 10000; i++)
        {
            UpdateRandomIndexList(indexList);

            var key = BuildByKey(this, indexList);

            ... // 忽略其他代码
        }
    }
    ... // 忽略其他代码
}
```

接着依然是一句代码，将 ElementList 里面，计算结果不等于 key 值的 Element 淘汰

```csharp
class Manager
{
    public void Start()
    {
        ... // 忽略其他代码
        var indexList = new int[3];
        for (int i = 0; i < 10000; i++)
        {
            UpdateRandomIndexList(indexList);

            var key = BuildByKey(this, indexList);

            ElementList.RemoveAll(element => BuildByKey(element, indexList) != key);

            ... // 忽略其他代码
        }
    }
    ... // 忽略其他代码
}
```

淘汰之后，依然就到了根据剩下存活的 Element 构建出来新的 Element 的步骤了。由于这次的 BuildKey 只有在元素创建的时候才被调用，这就意味着需要改造 Create 方法，让 Element 的在创建时调用 BuildKey 方法，修改之后的代码如下

```csharp
class Element
{
    public Element Create()
    {
        Element element = new Element(Random);
        foreach (var key in KeyList)
        {
            element.KeyList.Add(key);
        }

        element.BuildKey();

        return element;
    }
    ... // 忽略其他代码
}
```

而根据以上的 BuildKey 方法的代码，只是将 KeyList 序列生成到 10 个，显然不符合当前的需求。毕竟现在在创建的时候，就设置了 KeyList 内容了，这会让 BuildKey 方法啥都没有做。 继续优化 BuildKey 方法，让此方法可以将 KeyList 进行更改

```csharp
class Element
{
    public void BuildKey()
    {
        const int count = 10;
        while (KeyList.Count < count)
        {
            var key = Random.Next(Key.MaxKeyValue);
            KeyList.Add(new Key(key));
        }

        KeyList[Random.Next(count)] = new Key(Random.Next(Key.MaxKeyValue));
    }
    ... // 忽略其他代码
}
```

以上代码就是优化完成之后的 BuildKey 方法。大概映射的含义就是，子 Element 可以从前辈继承到 KeyList 知识，只是子 Element 要做出变化，变化就是修改 KeyList 中的数据。当前的变化只是修改一项而已

更改完成之后，即可开始跑代码。当前的代码的含义总的来说就是，在此世界里面，有一段世界序列，世界规则将根据世界序列计算出一个值，同样的世界规则也使用相同的算法应用在每个 Element 小人儿上，一旦发现两者计算结果不相同，那么 Element 小人儿将被淘汰。存活下来的 Element 小人儿将可以创建出下一代 Element 小人儿出来，创建的时候，可以将自己的知识传授给下一代。而下一代 Element 小人儿也不是全盘接受的，而是会经过自己的思考，优化从前辈继承到 KeyList 知识

只不过呢，在此世界里面，世界规则只是一个加法规则。而下一代 Element 小人儿的思考也只是一段随机数

尝试运行程序，可以看到这一次灭世的情况发生的次数比一开始的代码的多。灭世的情况就是在世界的某一代中，存活下来的 Element 小人儿的数量是 0 个。可以看到当前代码的规则让 Element 小人儿更加难以应对。但也同时发现了这样的规律，一旦开始有某个 Element 小人儿掌握了一定数量的正确的世界序列的值时，此 Element 小人儿将可以创建出更多更好存活的下一代 Element 小人儿

举个例子来说明运行的情况

假定世界序列就是从 0 到 9 的 10 个 Key 数值。某个 Element 掌握的序列就是 1020120120 序列。刚好一开始的 indexList 就是取前三个，这就让 Element 存活起来了。于是接下来 Element 将创建出下一代的 Element 出来。下一代的 Element 刚好修改的第 3 个（从0开始）序号为 4 的值，修改之后的是 1024120120 序列。下一轮世界取的 indexList 是 1 2 3 三个序号，世界序列计算结果是 1+2+3=6 的值。而原先的 Element 计算出来的是 0+2+0=2 被淘汰，而下一代的 Element 计算出来的是 0+2+4=6 存活。可以看到，尽管下一代的 Element 没有思考出正确的世界序列，然而在世界规则的处理下，依然还是可以存活的，只不过在偏离世界序列的情况下，可以存活的代数自然是有限的

这也能说明能存活下来的 Element 前辈掌握的世界序列是有一定的正确性的，但是下一代的 Element 是不知道前辈有哪些序列是正确的，甚至前辈的 KeyList 都是偏离世界序列的，只是刚好满足当前的世界规则而存活

在经过了世界大循环之后，可以再来看看存活下来的 Element 小人儿掌握世界序列的情况。这次的结果是绝大部分的 Element 小人儿掌握了正确的世界序列，之后少部分的 Element 小人儿掌握了错误的世界序列。这些少部分的掌握错误的世界序列的 Element 小人儿都是在最后几代世界循环里面创建的。也就是说活的足够久的 Element 小人儿都是掌握了正确的世界序列的

这也能映射到现实世界的一部分，能够存活下来的生物，或者人，都是掌握了一定的世界序列，也就是世界本源或者说世界的知识，或者说是符合世界规则。掌握的越多，或者说行为越靠近世界规则，就越能存活

此代码版本，放在 GitHub 上，可以从 [https://github.com/lindexi/lindexi_gd/commit/5f7d17b206b904db424d17fed6cef48eb0965496](https://github.com/lindexi/lindexi_gd/commit/5f7d17b206b904db424d17fed6cef48eb0965496) 获取全部代码

这个代码游戏到这里还没有结束，可以玩的还有很多。例如修改下一代的 Element 的思考逻辑，让下一代的 Element 更加有想法，想要更改更多的知识。在修改之后的代码世界，下一代的 Element 更加质疑前辈的知识，会更多的思考世界序列，而不是和之前的代码一样，只是修改世界序列里面的一项

修改之后的代码如下

```csharp
class Element
{
    public void BuildKey()
    {
        const int count = 10;
        while (KeyList.Count<count)
        {
            var key = Random.Next(Key.MaxKeyValue);
            KeyList.Add(new Key(key));
        }

        var updateCount = Random.Next(1, count);
        for (int i = 0; i < updateCount; i++)
        {
            KeyList[Random.Next(count)] = new Key(Random.Next(Key.MaxKeyValue));
        }
    }
    ... // 忽略其他代码
}
```

运行一下代码，可以看到此时运行的情况是，在很多代的世界大循环里面，即世界代数里，大量存在灭世的情况。因为下一代的 Element 小人儿思考越多，代表着继承到的前辈的知识越少。继承的知识越少，越难以存活。这也能说明前辈们的知识是有用的

此代码版本，放在 GitHub 上，可以从 [https://github.com/lindexi/lindexi_gd/commit/9bb128484abfa9fe6f5ba3ff66a91505fb3105b5](https://github.com/lindexi/lindexi_gd/commit/9bb128484abfa9fe6f5ba3ff66a91505fb3105b5) 获取全部代码

好了，今天的小测试就到这里。通过这个代码故事可以告诉咱，在这个代码世界里面，是符合天有五贼，见之者昌的道理，越接近世界序列的 Element 小人儿将越能存活。这个和当前的现实世界也差不多

在这个代码世界里面，前辈的知识是有用的，学习前辈的知识可以更好帮助存活，符合书籍是人类进步的阶梯这句话的道理。通过让前辈的知识传授给下一代，可以逐渐让整个世界在一代一代的循环里面存活的 Element 小人儿的数量越多

在此代码世界里面，前辈传授给下一代的知识也不一定是完全正确的，下一代需要进行质疑，因为前辈说不定活不过世界的下次迭代，前辈的知识只是刚好在当前的世界规则下是正确的，不代表着在下一次迭代的世界规则还是正确的。映射到现实世界，那就是需要质疑前辈的知识，前辈的知识放在前辈的时代是正确的，不代表在当前也是正确的。但是将前辈们的知识全抛掉，那自然也是不对的

现实世界里面也许存在世界序列和世界规则，然而现实世界里面可不会告诉大家，有多少个世界序列列表，世界序列的每个项的取值范围是多少，世界序列有多长等。试试在代码世界里面，让代码世界的世界序列的取值范围更大，或者是世界序列更长，试试看修改之后的灭世次数。同样现实世界不会告诉大家世界规则是什么，有多少个规则。现实世界的世界序列和世界规则更加复杂，这就让在现实世界进行探索世界的本质更加难。但是有一点是正确的，那就是学习前辈们的知识同时进行自己的思考，在一代代的努力下，可以不断靠近世界的本质

这个代码世界还有很多可以玩的，例如可以修改更有趣的世界规则，也就是 BuildByKey 方法。修改 Element 的思考方式等
