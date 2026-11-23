
# dotnet 在矩形堆里为新矩形找空位时如何避免扫描像素

在已有若干矩形的平面上放一个新矩形，怎么在不重叠的前提下找到一个合理位置，而且不做像素级或网格级扫描。

<!--more-->


<!-- 发布 -->
<!-- 博客 -->

本文内容由 AI 主导编写

## 背景

这个问题的形状比它看起来要常见。画布的自动添加容器只是一个外壳，同样的需求还包括：图表里给数据点加标注时的避让、地图 POI 标签排布、缩略图拼接、幻灯片自动排版、甚至代码里给错误提示气泡找位置。

抽象一下就是一句话：给定一个范围矩形、一个新矩形的宽高、一堆已有矩形，求新矩形左上角的坐标，使新矩形完全落在范围内，且与每个已有矩形至少保持固定间距。

朴素做法是扫描。把平面切成网格，逐格试放，撞了就换下一格。这条路会撞三面墙。

第一面墙是精度。真实数据里的可用缝隙可能很窄，比如两个障碍之间只剩 0.25 宽的空位。网格步长取大一点就直接漏掉这个位置，取小一点候选数量就爆掉。

第二面墙是顺序。人眼期待的结果是「从视口中心开始往外铺」，而网格编号很难自然表达这种顺序，你会需要额外的距离排序。

第三面墙是扩展。范围一旦按 1.2 倍扩大，网格数量按平方增长。

本文要讲的做法，是把连续平面里的搜索压成有限个候选坐标。核心只有两步：

1. 用膨胀障碍，把「矩形与矩形是否相交」变成「点是否落进禁区」
2. 用「可行点一定能平移到边界上」这条性质，把候选从无穷多个压到每轴 O(n) 个

后面的代码来自一个独立实验项目，项目本身包含算法库、单元测试和一个可视化界面，文末给出了获取整个项目的方式。

## 先把边界语义钉死

几何类型是最小的一层，但边界语义决定了后面所有的推导，所以先看它。

```csharp
public readonly record struct RectD(double X, double Y, double Width, double Height)
{
    public double Right => X + Width;
    public double Bottom => Y + Height;
    public PointD Center => new(X + Width / 2, Y + Height / 2);

    public bool Contains(RectD other) => other.X >= X && other.Y >= Y && other.Right <= Right && other.Bottom <= Bottom;

    public bool Overlaps(RectD other) => X < other.Right && Right > other.X && Y < other.Bottom && Bottom > other.Y;
}
```

两个方法用了不同的严格性组合：`Contains` 全部用 `>=`，边缘接触算被包含；`Overlaps` 全部用严格的 `<` 和 `>`，边缘接触不算重叠。

这个组合不是随手写的。它决定了允许的答案是「新矩形可以贴着范围边缘」，同时「新矩形可以贴着已有矩形边缘——只要间距够」。如果把 `Overlaps` 写成 `<=`，那么间距恰好等于要求值的位置会被判定为冲突，测试里那条「恰好 10 可用」的用例就会失败；如果把 `Contains` 写成 `>`，新矩形永远放不进视口，因为左上角刚好在视口左上角时会被判越界。

这里可以先记住一个结论：**在浮点几何里，闭区间包含和开区间重叠必须成对出现**，任何一边改成另一套严格性，都会让另一边的边界行为悄悄变化，而这类错误只会在「恰好等于」的输入上暴露。

## 第一步：把矩形碰撞变成点禁区

新矩形的位置只由左上角一个点决定，宽高是提前定死的。所以「找一个矩形的位置」其实等价于「找一个点的位置」。

既然目标是点，那就把每个已有矩形对左上角的限制也换算成一个点集。推导很短。

设新矩形左上角为 (x, y)、宽高为 w 和 h，某个已有矩形 E 的左上角为 (ex, ey)、宽高为 ew 和 eh。按照上面的 `Overlaps`，两者不重叠等价于下面四条之一成立：

```text
x >= ex + ew        新矩形整体在 E 右侧
x + w <= ex         新矩形整体在 E 左侧
y >= ey + eh        新矩形整体在 E 下方
y + h <= ey         新矩形整体在 E 上方
```

对不重叠取反，就得到重叠条件，也就是左上角的禁区：

```text
ex - w < x < ex + ew
ey - h < y < ey + eh
```

两个方向的开区间相乘，是一个挖掉的开矩形。它的含义很直观：新矩形要躲到 E 左边去，左上角最右只能到 E 左边再减掉自己的宽度；要躲到下边去，最下只能到 E 下边再减掉自己的高度。

**四个边界都是开区间端点**，因为贴着放不算重叠。这一点在后面会反复用到——它保证了候选点落在边界上时一定可行，不会自己踩线。

## 第二步：10 单位间距就是膨胀

有了禁区公式，「至少保持 10 单位间距」这件事根本不需要单独的判定逻辑。把障碍在四个方向各扩 10，再对新尺寸求禁区就行。

```csharp
private static RectD ExpandObstacle(RectD obstacle)
{
    RectD expanded = new(obstacle.X - MinimumMargin, obstacle.Y - MinimumMargin,
        obstacle.Width + 2 * MinimumMargin, obstacle.Height + 2 * MinimumMargin);
    ValidateArithmetic(expanded);
    if (obstacle.X - expanded.X < MinimumMargin || obstacle.Y - expanded.Y < MinimumMargin ||
        expanded.Right - obstacle.Right < MinimumMargin || expanded.Bottom - obstacle.Bottom < MinimumMargin)
        throw new OverflowException();
    return expanded;
}
```

扩张的是障碍，不是新矩形。选哪一种在数学上等价——把新矩形放大 20 和把障碍放大 20，得到的禁区是同一个——但工程上不等价：调用方最终要用原始的宽高去合成矩形，如果内部改了尺寸，返回值就必须带尺寸信息，接口就复杂了。所以这里对外只返回一个点。

真正值得看的是中间那段反向校验：扩张完，再去检查障碍到四个新边界的实际距离是否仍然不少于要求值。

正常数据下这段代码是纯粹的浪费，它永远不成立。但浮点不是实数。`1e300 - 10` 的结果仍然是 `1e300`，扩张在数值上什么也没做；`8e307 + 1.6e308` 直接变成无穷。这些情况下「扩张过」这个事实在 double 里不存在，如果继续往下算，算法会给出一个间距为零甚至为负的「合法」位置，而调用方完全看不出来。

所以这里的校验不是防御性冗余，而是**在浮点失效的那个点上主动崩溃**。它把一个静默的错误答案换成一个异常，代价是每次调用多四次减法。同样的思路也用在 `ValidateArithmetic` 上：

```csharp
private static void ValidateArithmetic(RectD rectangle)
{
    if (!double.IsFinite(rectangle.X) || !double.IsFinite(rectangle.Y) ||
        !double.IsFinite(rectangle.Right) || !double.IsFinite(rectangle.Bottom) ||
        rectangle.Right <= rectangle.X || rectangle.Bottom <= rectangle.Y)
        throw new OverflowException();
}
```

`Right <= X` 这一条查的是宽度被吃掉。当坐标接近 double 上限时，`X + Width` 可能精确等于 `X`，矩形宽度在数值上变成零。这时候几何意义已经消失，但后续的禁区计算会得出「任何位置都可行」，因为禁区宽度也是零。提前拦在这里，比让错误答案流到界面上要好。

## 第三步：连续平面里为什么只需枚举边界

这是整个算法最关键的一步，也是最容易讲不清的一步。问题是：可行域是「一个矩形挖掉 O(n) 个开矩形」之后的形状，点数无穷多，凭什么只看有限几个？

答案是一条单调性性质。

从任意一个可行点出发，保持 y 不动，把 x 一直往左推。对每个禁区 i，记它的左右边界为 lo(i) 和 hi(i)，可行条件是 `x <= lo(i)` 或者 `x >= hi(i)`，二者必居其一，因为出发点可行。

现在把 x 减小。如果对某个 i 原本满足的是 `x <= lo(i)`，减小之后仍然满足，没有风险。如果原本满足的是 `x >= hi(i)`，那么 x 可以一路减到 `hi(i)` 都没有风险——因为禁区是开区间，(lo, hi) 之外都是安全的，`x = hi(i)` 恰好还可行。再往下减就进入禁区了，所以停在 `hi(i)`。

把所有禁区的最紧下界取出来，x 可以单调地降到这个值，途中每一步都可行。这个终点要么是某个禁区的左边界 `lo(i)`，要么是范围矩形的左边界。

关键在「单调」两个字。x 往左推的时候，所有关于 x 的约束都是「不超过某个上界」的形式，所以推的过程只能让条件变宽松或者刚好触底，不会突然违约。这就是可以独立降维的原因：x 方向和 y 方向的约束在这个意义上是解耦的，可以先压 x，再压 y，两次都保持可行。

于是结论出来：**如果存在可行解，就存在一个可行解，其横坐标是某个禁区左边界或范围左边界，其纵坐标是某个禁区下边界或范围下边界。**

候选集从无穷多降到每轴 O(n) 个。这个思路在几何算法里叫关键坐标枚举，也可以理解为「所有空位都可以挪到边界上，所以中间的点没有信息量」。

代码就是把这个候选集造出来：

```csharp
private static int Events(Span<double> buffer, ReadOnlySpan<RectD> obstacles, double length,
    double center, double min, double max, bool vertical, out int split)
{
    int count = 0;
    for (int i = 0; i < obstacles.Length; i++)
    {
        RectD obstacle = obstacles[i];
        buffer[count++] = Difference(vertical ? obstacle.Y : obstacle.X, length);
        buffer[count++] = vertical ? obstacle.Bottom : obstacle.Right;
    }
    buffer[count++] = center;
    buffer[count++] = min;
    buffer[count++] = max;
```

每个障碍贡献两个候选，合计 `2n` 个，再加上中心、范围两端，正好 `2n+3`。这个数字决定了调用方要预分配的缓冲区大小，也解释了为什么有两个等长缓冲区：

```csharp
double[] rowBuffer = new double[checked(2 * obstacles.Length + 3)];
double[] columnBuffer = new double[rowBuffer.Length];
```

`Difference(obstacle.Y, length)` 算的就是禁区下边界，也就是「新矩形挤到障碍下方时最靠上能到哪」；`obstacle.Bottom` 是禁区上边界，也就是「挤到障碍上方时最靠下能到哪」。缓冲区用 `Span` 接收并在原地读写，没有 `List<double>` 的扩容。

构造完还要过三道工序：

```csharp
int filtered = 0;
for (int i = 0; i < count; i++)
{
    double value = buffer[i];
    if (value >= min && value <= max) buffer[filtered++] = value;
}
buffer = buffer[..filtered];
buffer.Sort();
int unique = 0;
split = 0;
for (int i = 0; i < buffer.Length; i++)
{
    double value = buffer[i];
    if (unique > 0 && value == buffer[unique - 1]) continue;
    buffer[unique++] = value;
    if (value < center) split++;
}
return unique;
```

过滤把越界候选去掉，排序为后来的双向遍历做准备，去重是因为多个障碍经常贡献同一个边界值——比如几个障碍左对齐。`split` 是额外副产品：去重时顺便数出有多少个候选严格小于中心，这个数字就是下一步的分界线。

注意这三道工序全部是原地进行，复用的是同一块 `2n+3` 缓冲区。后面会讲到这为什么值得单独写测试守住。

## 第四步：把「中心优先、上下交替」折叠进遍历

候选集是集合，决定最终结果的是遍历顺序。这里要求的顺序有三层：

- 中心行优先
- 中心以下是下1、下2……按 Y 递增；中心以上是上1、上2……按 Y 递减
- 实际尝试顺序是中心、下1、上1、下2、上2……

第一反应可能是把上下两半拆成两个数组再合并，但那样要额外分配，还要处理一侧先耗尽。排序后的数组本身已经满足「下半递增、上半递减」的天然结构——只要从 `split` 这个分界处向两边走，下半向右、上半向左，两边取到的数字正好就是下1、下2和上1、上2。

```csharp
int above = rowSplit - 1;
int below = rowSplit;
bool centerPending = below < rowCount && rowBuffer[below] == center.Y;
bool nextBelow = true;
while (above >= 0 || below < rowCount)
{
    double y;
    if (centerPending)
    {
        y = rowBuffer[below++];
        centerPending = false;
    }
    else
    {
        y = (nextBelow && below < rowCount) || above < 0
            ? rowBuffer[below++] : rowBuffer[above--];
        nextBelow = !nextBelow;
    }
```

`above` 和 `below` 就是两个游标，一个向下走一个向上走，全程只占两个 `int`。

`centerPending` 处理的是中心行落在下半区这个细节。中心坐标本身也在候选集里，去重排序之后它必然位于 `split` 这个位置，会被算进下半区，所以第一轮如果直接按下1处理，中心行会被跳过。这里先把下半区首项和中心比较一次，相等就先输出它，再把游标推过去。

中间的布尔表达式值得逐项读一遍：想取下方、并且下方确实还有候选，就取下方；否则如果上方已经耗尽，也必须取下方；剩下的情况取上方。三个分支压成一行，就把「一侧耗尽后继续另一侧」这个规则吸收进了循环条件，不需要额外的状态标志。

行序确定之后，每行的列也要按「先右后左」走。这里的实现更精简，把两段方向相反的遍历折叠成了同一个 `for`：

```csharp
for (int column = 0; column < columnCount; column++)
{
    int index = column < columnCount - columnSplit
        ? columnSplit + column : columnCount - 1 - column;
```

`columnSplit` 是候选 X 里严格小于中心的数量，`columnCount - columnSplit` 就是右侧候选的个数。前半段 `column` 从 0 涨到右侧个数减一，索引映射到 `columnSplit + column`，也就是右半区从小到大；一旦 `column` 越过这个数，索引改成 `columnCount - 1 - column`，取值从右半区末尾往前退，正好衔接在左半区的最大项上，继续递减。

一个循环、一个三元表达式，同时表达了「先搜完右侧」和「左侧从大到小」两个要求。代价是索引可比 `column` 读起来费点劲，所以这种情况必须配注释说明语义，否则后人只看到 `columnCount - 1 - column` 会以为是倒序遍历整个数组。

## 第五步：候选的最终验证

走到这一步，候选点已经不多，剩下的就是老老实实检查。

```csharp
RectD candidate = new(columnBuffer[index], y, size.Width, size.Height);
ValidateArithmetic(candidate);
if (!range.Contains(candidate)) continue;
bool overlaps = false;
for (int i = 0; i < obstacles.Length; i++)
{
    if (candidate.Overlaps(obstacles[i]))
    {
        overlaps = true;
        break;
    }
}
if (!overlaps) return new(candidate.X, candidate.Y);
```

两个检查缺一不可。范围检查是因为候选坐标虽然在 `[min, max]` 内，但那是**左上角**的范围；对某一轴的候选来说，另一轴的取值可能让整块矩形探出范围。碰撞检查是因为禁区候选只保证「对贡献这个坐标的那个障碍不重叠」，对其他障碍没有任何承诺。

复杂度在这里落地：外层行 O(n)、内层列 O(n)、每个候选检查 O(n)，最坏 O(n³)，空间 O(n)。这个账需要明确写进文档——它意味着算法在障碍上千时不该被调用，而不是「反正能跑」。

## 第六步：所有候选都失败之后

有一种情况必须处理：范围本身装不下新矩形，或者所有候选都被占满。

```csharp
double nextWidth = width * 1.2;
double nextHeight = height * 1.2;
if (!double.IsFinite(nextWidth) || !double.IsFinite(nextHeight) || nextWidth <= width || nextHeight <= height)
    throw new OverflowException();
width = nextWidth;
height = nextHeight;
range = new(center.X - width / 2, center.Y - height / 2, width, height);
```

宽高分别乘 1.2，中心固定，所以范围是向四周均匀膨胀的。保留中心是因为扩展后的结果要延续「从中心往外铺」的视觉习惯，如果扩展到右上角去了，前几轮的努力会白费。

`nextWidth <= width` 这一条看着别扭，但它和前面 `Right <= X` 是同一个问题的另一面：当 `width` 大到 `width * 1.2` 在 double 里等于 `width` 时，循环会永远转下去。判断「这次扩展有没有真的变大」，是无限循环唯一可靠的出口。

另外，扩展只发生在这个函数内部的逻辑范围上，函数不修改视口、不修改已有矩形、也不调整视图缩放。谁来决定要不要把界面缩放回去，是调用方的事。这条边界划清楚之后，函数才可能在测试里被反复调用而不产生副作用。

## 怎么证明候选枚举没有漏解

前面第三步的推导是纸面证明。纸面证明会写错，所以还需要机器验证。

思路很简单：找一个**暴力实现**当 oracle。既然网格扫描会漏解、会慢，那就在一个足够小的场景里让网格扫描跑起来，把它当成标准答案。

```csharp
RectD viewport = new(0, 0, 23, 23);
RectD[] cells = Enumerable.Range(0, 9).Select(i => new RectD(i % 3 * 11, i / 3 * 11, 1, 1)).ToArray();
RectD[] candidates = Enumerable.Range(0, 23 * 23).Select(i => new RectD(i % 23, i / 23, 1, 1)).ToArray();
bool[] matches = Enumerable.Range(0, 512).Select(mask =>
{
    RectD[] obstacles = cells.Where((_, index) => (mask & (1 << index)) != 0).ToArray();
    bool gridHasSpace = candidates.Any(candidate => obstacles.All(obstacle => HasMinimumGap(candidate, obstacle)));
    PointD result = Placement.Find(viewport, new(1, 1), obstacles);
    RectD rectangle = new(result.X, result.Y, 1, 1);
    return gridHasSpace == viewport.Contains(rectangle) &&
        obstacles.All(obstacle => HasMinimumGap(rectangle, obstacle));
}).ToArray();
```

场景参数是精心凑的：视口 23×23，9 个 1×1 的小格子按 11 的间距摆成 3 行 3 列。11 = 1 + 10，正好是一个格子加上间距，所以每个格子把自己的 11×11 区域完全堵死，而格子之间的剩余缝隙恰好是争议地带。整体尺寸又小到 23×23 可以逐像素穷举。

9 个格子取子集，`2^9 = 512` 种布局，全部跑一遍。断言的是一个等价关系：网格穷举认为有位置，算法必须也给出一个合法位置；网格穷举认为没位置，算法必须报告失败（在测试里表现为异常或返回位置越界）。

这个断言两个方向都有价值。`gridHasSpace == true` 但算法失败，说明候选集漏了某个边界；`gridHasSpace == false` 但算法给出了位置，说明可行性检查有漏洞。前者是第三步推导的错误，后者是第五步检查的错误，两者被这一个用例同时守住。

`HasMinimumGap` 是测试内部独立写的间距判定，不复用生产代码的 `Overlaps`。这点很重要：如果 oracle 也调用被测试的实现，那么实现错了 oracle 会跟着错，测试就退化成同义反复。

再往下还有一条更细的用例，专门验证分数窄缝不被跳过：

```csharp
RectD[] obstacles = [new(0, 0, 39.125, 100), new(59.375, 0, 40.625, 100)];
Assert.AreEqual(new PointD(49.125, 50), Placement.Find(new(0, 0, 100, 100), new(0.25, 10), obstacles));
```

两个障碍之间留了 20 宽的空隙，扣掉两边的 10 间距，实际可用宽度是 0。25。这类数据在网格扫描下几乎必然被漏掉，因为任何大于 0.25 的步长都会跨过去。

## 怎么守住「不额外分配」

算法每轮扩展都复用同一块缓冲区，这个设计如果被后来的人改回 `new List<double>()`，结果依然正确，只是悄悄开始分配。性能回归测试最怕这种「功能正确但退化了」的改动。

.NET 里能做这件事的 API 是 `GC.GetAllocatedBytesForCurrentThread`，它返回当前线程累计分配字节数，做差就是这段时间的分配量：

```csharp
private static long Measure<T>(Func<T> find, int iterations)
{
    find();
    long start = GC.GetAllocatedBytesForCurrentThread();
    for (int i = 0; i < iterations; i++) find();
    return GC.GetAllocatedBytesForCurrentThread() - start;
}
```

泛型参数不是装饰。这段代码要同时测新旧两个实现：旧实现的返回值是引用类型，新实现返回 `PointD` 值类型，如果写成 `Func<object>`，每次调用都会把 `PointD` 装箱，测出来的分配量就是测试自己制造的，跟被测量对象无关。用 `Func<T>` 泛型，装箱不会发生。

测试里还有两个容易忽略的细节。一是测量前先跑一次预热，把方法首次调用的 JIT 和静态初始化开销排除掉；二是断言用的是宽松的线性预算，而不是精确字节数：

```csharp
const long budgetPerCall = 96;
long allowed = 1024 + budgetPerCall * count;
```

`96 * n` 覆盖障碍数组的复制和两个缓冲区的线性增长，`1024` 是常数项。为什么不用精确值？因为精确值依赖运行时版本、JIT 决策甚至处理器架构，写死之后换台机器就红。宽松预算只守一件事：**数量级不能从 O(n) 变成别的形状**。有人把缓冲区改成每轮新建，实测值会立刻从几十字节跳到上千字节，预算必然击穿。

测量结果通过 `TestContext` 输出到 TRX 报告，所以即使测试全绿，也能翻出每次跑的实际数字做纵向对比。测试本身不断言耗时——耗时在 CI 机器上没有意义，分配量有。

## 这套方法什么时候不该用

推导再顺，也有边界。以下几种情况应该换思路：

- 障碍数量到几千以上。O(n³) 会直接压垮交互，需要换成网格空间索引或者扫描线维护空闲矩形集合。
- 需要三维、支持旋转、或者形状不是轴对齐矩形。禁区的形式会完全不同，第三步的独立性论证也随之失效。
- 需要「最优」而不是「够用」。这里给出的第一个可行解，不是面积最省或移动距离最小的解，要最优就得进优化算法。
- 需要整齐的视觉效果。边界候选天然产生参差不齐的位置，如果要让结果对齐到栅格，正确做法是给候选集加吸附步长，而不是事后取整——事后取整会破坏可行性。

对本文的场景来说，障碍上限是几十个，交互频率是一次点击一次计算，O(n³) 完全够，所以选择了「最简单、最容易证明正确」的实现，而不是空间索引。这个取舍本身也是需要写进文档的一部分：把复杂度边界和适用规模明确写出来，比留一句「性能还行」有用得多。

## 小结

把整个过程压缩成四步，可以套到别的「在已有形状之间找位置」的问题上：

1. **把尺寸问题变点问题**：位置只由锚点决定时，把每个已有形状按新形状尺寸反向膨胀，碰撞判定就变成锚点是否落进禁区。
2. **把连续问题变候选问题**：证明可行点可以单调滑动到某个边界上，候选就从无穷多变成每轴 O(n)。
3. **把顺序要求变索引问题**：候选排好序之后，中心优先和双向交替都可以用两个游标原地实现，不需要合并数组。
4. **把正确性变可验证问题**：在小规模场景里用暴力扫描当 oracle，把纸面推导变成可执行的等价断言。

每一步单独看都不是什么高深技巧，但串起来之后，一个原本只能靠「扫一遍试试」的问题，变成了一个可以做证明、可以做差分测试、可以守住分配量的确定算法。

本文代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/bfaf15ddd0b942d0ed52853bd24183fe6dcb359b/Workbench/FantasticCanvasPlacementExperiment) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/bfaf15ddd0b942d0ed52853bd24183fe6dcb359b/Workbench/FantasticCanvasPlacementExperiment) 上，可以使用如下命令行拉取代码。我整个代码仓库比较庞大，使用以下命令行可以进行部分拉取，拉取速度比较快

先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin bfaf15ddd0b942d0ed52853bd24183fe6dcb359b
```

以上使用的是国内的 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码，将 gitee 源换成 github 源进行拉取代码。如果依然拉取不到代码，可以发邮件向我要代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin bfaf15ddd0b942d0ed52853bd24183fe6dcb359b
```

获取代码之后，进入 Empty/Workbench/FantasticCanvasPlacementExperiment 文件夹，即可获取到源代码

更多技术博客，请参阅 [博客导航](https://blog.lindexi.com/post/%E5%8D%9A%E5%AE%A2%E5%AF%BC%E8%88%AA.html )




<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。