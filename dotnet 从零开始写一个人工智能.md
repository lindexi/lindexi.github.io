# dotnet 从零开始写一个人工智能

本文将不使用任何人工智能框架，只用简单的 dotnet 的类，自己搭建一个人工智能网络。本文适合小伙伴跟着一步步写

<!--more-->
<!-- CreateTime:2020/1/30 16:55:33 -->

<!-- 发布 -->

特别感谢[老马的程序人生](https://www.jianshu.com/u/16f2e309c8ec)的帮助，本文有大量代码都是从[如何利用 C# 抽象神经网络模型](https://www.jianshu.com/p/db4cef1c0484)抄的

在人工智能模型有不同的问题可以选用不同的模型，本文主要写一个 BP 网络用于做分类，也就是写出一个简单的多分类人工智能和一个模拟二进制 与 计算和 或 计算。请不要认为本文会告诉大家如何写一个会和你聊QQ的人工智能，这里的人工智能其实也就是一个工具，和想象的智能差距有点大。本文的人工智能只能做对数值输入进行分类或实现模拟二进制计算

在一个人工智能模型，可以将人工智能模型作为网络模型建立，一个人工智能模型是一个网络模型，一个网络里面会有很多层，每一层有很多元。本文将从小到大进行定义，先从元定义，然后再从层定义，最后定义网络

在人工智能模型里面，最小单元都是神经元。一个神经元可以收到多个输入，而只有一个输出。在代码里面，将输入和输出的值都定义为double值。而本文写的神经元是固定输入数量，也就是在神经元对象创建的时候需要告诉这个神经元可以收到多少个数量的输入

在神经元里面将会对每个输入添加一个权值，在神经网络的每个元可以收到多个输入，而对每个输入需要使用不同的权值计算

最简单的神经元就是将每个输入的值乘以一个权值然后加起来然后输出。当然稍微复杂一点的是加起来之后需要加一个阈值然后调用激活函数计算输出

先忽略元的计算，定义元的数据结构

```csharp
    /// <summary>
    /// 神经元
    /// </summary>
    public abstract class Neuron
    {
        /// <summary>
        /// 随机数生成器
        /// </summary>
        public static Random Rand { get; set; } = new Random();

        /// <summary>
        /// 随机数范围
        /// </summary>
        public static DoubleRange RandRange { get; set; } = new DoubleRange(0.0f, 1.0f);

        /// <summary>
        /// 多输入
        /// </summary>
        public int InputsCount { get; }

        /// <summary>
        /// 单输出
        /// </summary>
        public double Output { get; protected set; } = 0.0;

        /// <summary>
        /// 权值数组，每个输入对应一个权值，也就是 InputsCount 的数量和 Weights 元素数相同
        /// </summary>
        /// 在神经网络的每个元可以收到多个输入，而对每个输入需要使用不同的权值计算。此时对应每个输入一个权值，而输出只有一个
        public double[] Weights { get; }

        /// <summary>
        /// 构造函数
        /// </summary>
        /// <param name="inputs">表示有多少个输入</param>
        protected Neuron(int inputs)
        {
            // 输入的数量不能小于1的值
            InputsCount = Math.Max(1, inputs);
            Weights = new double[InputsCount];
            Randomize();
        }

        /// <summary>
        /// 初始化权值
        /// </summary>
        public virtual void Randomize()
        {
            for (int i = 0; i < InputsCount; i++)
            {
                // 创建在 RandRange.Max 和 RandRange.Min 范围内的随机数
                Weights[i] = RandRange.GetRan();
            }
        }

        /// <summary>
        /// 对当前传入的输入计算输出的值
        /// </summary>
        /// <param name="input">输入的值的数量要求和 InputsCount 相同</param>
        /// <returns></returns>
        public abstract double Compute(double[] input);
    }
```

上面的模型感谢[老马的程序人生](https://www.jianshu.com/u/16f2e309c8ec)大佬提供，只是定义了基础数据结构，而计算方法作为抽象方法。在 Compute 方法接受多个输入，然后有一个 double 输出

在 Randomize 方法给了权值数组一些随机值，其实有一句话是人工智能和随机猜是差不多的。本文下面将写一个随机给权值的训练方法

上面的 DoubleRange 是自己定义的，用来创建范围内的随机数，只需要看代码就知道是如何做的

```
    public class DoubleRange
    {
        public DoubleRange(double a, double b)
        {
            Max = Math.Max(a, b);
            Min = Math.Min(a, b);

            Length = Max - Min;
        }

        public double Length { get; }
        public double Max { get; }
        public double Min { get; }

        /// <summary>
        /// 返回在 Min 和 Max 的随机数
        /// </summary>
        /// <returns></returns>
        public double GetRan()
        {
            return Rand.NextDouble() * Length + Min;
        }

        /// <summary>
        /// 随机数生成器
        /// </summary>
        private static Random Rand { get; set; } = new Random();
    }
```

接下来定义一个 ActivationNeuron 继承 Neuron 这里定义了阈值和激活函数

```csharp
    /// <summary>
    /// 神经元
    /// </summary>
    public class ActivationNeuron : Neuron
    {
        /// <summary>
        /// 阈值
        /// </summary>
        public double Threshold { get; set; } = 0.0;

        /// <summary>
        /// 激活函数
        /// </summary>
        public IActivationFunction ActivationFunction { get; set; }

        /// <summary>
        /// 构造函数
        /// </summary>
        /// <param name="inputs">输入个数</param>
        /// <param name="function">激活函数</param>
        public ActivationNeuron(int inputs, IActivationFunction function)
            : base(inputs)
        {
            ActivationFunction = function;
        }

        /// <summary>
        /// 初始化权值阈值
        /// </summary>
        public override void Randomize()
        {
            base.Randomize();
            Threshold = RandRange.GetRan();
        }

        /// <summary>
        /// 计算神经元的输出
        /// </summary>
        /// <param name="input"></param>
        /// <returns></returns>
        public override double Compute(double[] input)
        {
            if (input.Length != InputsCount)
                throw new ArgumentException("输入向量的长度错误。");

            double sum = 0.0;
            for (int i = 0; i < Weights.Length; i++)
            {
                sum += Weights[i] * input[i];
            }

            sum += Threshold;
            double output = ActivationFunction.Function(sum);
            Output = output;
            return output;
        }
    }
```

计算的方法是将多个输入的每个输入乘以权值加起来，然后加上阈值，接下来放入激活函数计算输出。为什么需要激活函数？原因是计算的 sum 的值需要输出到下一层需要将 sum 的值处理，如本文需要让每一层的输入的值都是 `1` 和 `0` 而因为每一层的输出会作为下一层的输入，所以需要将 sum 值计算为 `1` 和 `0` 也就是通过 阈值函数 将一个值按照是否大于等于 0 分为 `1` 和 `0` 两个值

那么 阈值函数 的定义是什么，阈值就是临界值，函数的目的是大于这个临界值会怎么样，小于这个临界值会怎么样。本文只是让大于等于 0 输出为 `1` 否则输出 0 这个值

```
    /// <summary>
    /// 阈值函数
    /// </summary>
    public class ThresholdFunction : IActivationFunction
    {
        public double Function(double x)
        {
            return (x >= 0) ? 1 : 0;
        }

        public double DerivativeX(double x)
        {
            return 0;
        }

        public double DerivativeY(double y)
        {
            return 0;
        }
    }

```

激活函数的定义是

```csharp
    /// <summary>
    /// 对激活函数的抽象
    /// 所有与神经元一起使用的激活函数，都应该实现这个接口，这些函数将神经元的输出作为其输入加权和的函数来计算
    /// </summary>
    public interface IActivationFunction
    {
        /// <summary>
        /// 激活函数
        /// </summary>
        /// <param name="x"></param>
        /// <returns></returns>
        double Function(double x);

        /// <summary>
        /// 求x点的导数
        /// </summary>
        /// <param name="x"></param>
        /// <returns></returns>
        double DerivativeX(double x);

        /// <summary>
        /// 求y点的导数
        /// </summary>
        /// <param name="y"></param>
        /// <returns></returns>
        double DerivativeY(double y);
    }
```

和 阈值函数 相应的激活函数还有符号函数等，符号函数就是按照一定的范围，将输入转换为 `1` 或 `-1` 因为负数叫符号，这也就是符号函数的名字。详细定义请看 [符号函数](https://baike.baidu.com/item/符号函数/3198406)

```
    /// <summary>
    /// 符号函数
    /// </summary>
    public class SignFunction : IActivationFunction
    {
        public double Function(double x)
        {
            return x >= 0 ? 1 : -1;
        }

        public double DerivativeX(double x)
        {
            return 0;
        }

        public double DerivativeY(double y)
        {
            return 0;
        }
    }

```

也就是在接受多个输入，对多个输入乘以权值加起来，再加上阈值，放入激活函数，计算后输出。这就是最简单的元的定义

定义完成了元，接下来就是定义层的概念，每一层可以有多个元，每一层可以收到上一层的数据。而第一层叫输入层，输入层将会接受用户的输入。最后一层叫输出层，输出层的值将会作为输出。简单的网络只需要一个元，一个元作为一层，而这个网络也只有一层。这一层只有一个元，是输入层也是输出层。这是最简单的模型，也就是本文接下来告诉大家的模型。也就是元是核心，只有一个元也能做出人工智能

```csharp
    /// <summary>
    /// 对神经网络层的抽象，代表该层神经元的集合
    /// </summary>
    public abstract class Layer
    {
        /// <summary>
        /// 该层神经元的个数
        /// </summary>
        protected int NeuronsCount { set; get; }

        /// <summary>
        /// 该层的输入个数
        /// </summary>
        public int InputsCount { get; }

        /// <summary>
        /// 该层神经元的集合
        /// </summary>
        public Neuron[] Neurons { get; }

        /// <summary>
        /// 该层的输出向量
        /// </summary>
        public double[] Output { get; protected set; }

        /// <summary>
        /// 神经网络层的抽象
        /// </summary>
        /// <param name="neuronsCount">该层神经元的个数</param>
        /// <param name="inputsCount">该层的输入个数</param>
        /// 每一层的神经元的个数不同，同一层的每个神经元的输入个数相同，但不同层的输入个数可以不同
        protected Layer(int neuronsCount, int inputsCount)
        {
            InputsCount = Math.Max(1, inputsCount);
            NeuronsCount = Math.Max(1, neuronsCount);
            Neurons = new Neuron[NeuronsCount];
        }

        /// <summary>
        /// 计算该层的输出向量
        /// </summary>
        /// <param name="input"></param>
        /// <returns></returns>
        public virtual double[] Compute(double[] input)
        {
            double[] output = new double[NeuronsCount];
            for (int i = 0; i < Neurons.Length; i++)
            {
                output[i] = Neurons[i].Compute(input);
            }

            Output = output;
            return output;
        }

        /// <summary>
        /// 初始化该层神经元的权值
        /// </summary>
        public virtual void Randomize()
        {
            foreach (Neuron neuron in Neurons)
            {
                neuron.Randomize();
            }
        }
    }
```

定义的层需要知道输入的数量，而层里每个元都会被定义相同数量的输入。此时的连接和全连接差不多，也就是每个元都接受到相同的输入。其实这样的定义对于某个元只需要特定的几个输入也是可以实现的，因为每个元会对每个输入一个权值，如果设置某个输入的权值为 0 那么相当于放弃这个输入。这样就可以做到某个元只接受特定的几个输入而不是收到所有的输入。而为什么一些高级的模型不会让同一层的所有元收到的输入相同？刚才不是说可以让元自己控制放弃哪些输入，原因是虽然可以让元自己控制放弃一些输入，但是这样做的效率比较低，高级的模型需要提升效率，本文这里无视所以可以使用每一层的所有元收到相同的输入

本文这里使用的模型是每一层的元数量不可变，在定义层时就知道这一层有多少元。这样的模型功能会比较差，但是作为入门的博客，这样的定义差不多可以使用了

每一个元只有一个输入，所以每一层的输出数量和每一层的元数量相同

接下来定义 ActivationLayer 作为实际的神经网络层，其实从代码可以看到连个类可以合并在一起，只是[老马的程序人生](https://www.jianshu.com/u/16f2e309c8ec)大佬作为两篇博客，我这里也就跟着他写了

```
    public class ActivationLayer : Layer
    {
        /// <summary>
        /// 神经网络层
        /// </summary>
        /// <param name="neuronsCount">该层神经元的个数</param>
        /// <param name="inputsCount">该层的输入个数</param>
        /// <param name="function">激活函数</param>
        public ActivationLayer(int neuronsCount, int inputsCount, IActivationFunction function)
            : base(neuronsCount, inputsCount)
        {
            for (int i = 0; i < Neurons.Length; i++)
                Neurons[i] = new ActivationNeuron(inputsCount, function);
        }
    }
```

一个网络可以包含多个层

```csharp
   /// <summary>
    /// 对神经网络结构的抽象，它表示神经元层的集合
    /// </summary>
    public abstract class Network
    {
        /// <summary>
        /// 网络层的个数
        /// </summary>
        protected int LayersCount { set; get; }

        /// <summary>
        /// 网络输入的个数
        /// </summary>
        public int InputsCount { get; }

        /// <summary>
        /// 构成网络的层
        /// </summary>
        public Layer[] Layers { get; }

        /// <summary>
        /// 网络的输出向量
        /// </summary>
        public double[] Output { get; protected set; }

        /// <summary>
        /// 构造函数
        /// </summary>
        /// <param name="inputsCount">输入的个数</param>
        /// <param name="layersCount">网络层的数量</param>
        protected Network(int inputsCount, int layersCount)
        {
            InputsCount = Math.Max(1, inputsCount);
            LayersCount = Math.Max(1, layersCount);
            Layers = new Layer[LayersCount];
        }

        /// <summary>
        /// 计算网络的输出
        /// </summary>
        /// <param name="input">要求输入的元素数和 InputsCount 网络输入的个数相同</param>
        /// <returns></returns>
        public virtual double[] Compute(double[] input)
        {
            // 第一层用于输入，将输入层作为传输
            double[] courier = input;

            for (int i = 0; i < Layers.Length; i++)
            {
                // 第一层的输出作为第二层的输入
                // 所以输入是 courier 而返回的输出作为下一层的输入
                courier = Layers[i].Compute(courier);
            }
            // 最后一层的输出作为网络输出
            Output = courier;
            return courier;
        }

        /// <summary>
        /// 初始化整个网络的权值
        /// </summary>
        public virtual void Randomize()
        {
            foreach (Layer layer in Layers)
            {
                layer.Randomize();
            }
        }
    }
```

请看一下代码里面的注释

实现一个网络

```csharp
    public class ActivationNetwork : Network
    {
        /// <summary>
        /// 神经网络
        /// </summary>
        /// <param name="function"></param>
        /// <param name="inputsCount"></param>
        /// <param name="neuronsCount">指定神经网络每层中的神经元数量</param>
        public ActivationNetwork(IActivationFunction function, int inputsCount, params int[] neuronsCount)
            : base(inputsCount, neuronsCount.Length)
        {
            // neuronsCount 指定神经网络每层中的神经元数量。
            for (int i = 0; i < Layers.Length; i++)
            {
                Layers[i] = new ActivationLayer(neuronsCount[i],
                    // 每个神经元只有一个输出，上一层有多少个神经元也就有多少个输出，也就是这一层需要有多少输入
                    (i == 0) ? inputsCount : neuronsCount[i - 1],
                    function);
            }
        }
```

上面代码的实现有些诡异，原因是我的参数没有写好。在 ActivationNetwork 的最后一个参数是一个数组，指定神经网络每层中的神经元数量。也就是输入 `[1,2]` 表示有两层，其中第一层有 1 个神经元，第二层有两个。输入 `[1,1,5]` 表示有三层

而根据人工智能的教程，第一层是输入层，也就是 `i == 0` 设置这一层的输入为用户输入数量。从第二层开始，每一层的输入数量为上一层的输出数量

定义完成了人工智能模型，一个模型不会自动运行，还需要定义一个训练方法。作为可以自己学习的人工智能，学习方法可以分为监督学习和无监督学习，在代码里面我用[老马的程序人生](https://www.jianshu.com/u/16f2e309c8ec)的说法非监督学习。这两个方法的不同在于监督学习是我知道输入内容和结果，我将输入放入模型，对比模型输出的值和我知道的结果，按照模型输出的值和我知道的结果的误差反馈给模型，让模型修改参数，如修改权值参数。而无监督学习是我也不知道结果，这个比较难理解，详细请看[监督学习](https://baike.baidu.com/item/监督学习/9820109?fr=aladdin)和[无监督学习](https://baike.baidu.com/item/无监督学习/810193) 或 [小白都看得懂的监督学习与无监督学习](https://www.jianshu.com/p/682c88cee5a8) 本文用到的是监督学习

```csharp
    /// <summary>
    /// 对监督学习的抽象，这个接口描述了所有监督学习算法应该实现的方法
    /// </summary>
    /// 监督学习和下面的非监督学习的不同在于，非监督学习只需要给输入，不需要给输出，也就是在训练是不需要知道结果，而监督学习是需要知道输入是什么对应输出是什么，相对于非监督学习，理解监督学习比较简单
    public interface ISupervisedLearning
    {
        /// <summary>
        /// 单样本训练
        /// </summary>
        /// <param name="input"></param>
        /// <param name="output"></param>
        /// <returns></returns>
        double Run(double[] input, double[] output);

        /// <summary>
        /// 多样本训练
        /// </summary>
        /// <param name="input"></param>
        /// <param name="output"></param>
        /// <returns></returns>
        double RunEpoch(double[][] input, double[][] output);
    }
```

这里的输入分为多样本训练和单样本训练也就是我给一堆数据就是多样本训练。从方法参数可以看到输入的都是二维数组，当然这里说二维数组是不对的，应该是数组的数组。从单样本训练方法可以看到每个数据都是输入是一个 double 数组，而输出也是一个 double 数组，那么多个输入和多个输出就是数组的数组

刚才也有说到，人工智能和随机猜是一样的，在人工智能的训练很重要的是反馈，也就是我告诉人工智能说算错了，他应该如何修改参数？这部分看起来有点难，假设这个人工智能我告诉他算错了，他就随机修改他的参数，这就是本文的 Slow 训练方法，这个方法只是慢，和其他训练方法差不多

```csharp
    public class SlowLearning : ISupervisedLearning
    {
        public SlowLearning(ActivationNetwork network)
        {
            _network = network;
        }

        /// <summary>
        /// 神经网络
        /// </summary>
        private readonly ActivationNetwork _network;

        private DoubleRange RandRange { get; } = new DoubleRange(-1.0, 1.0);

        /// <summary>
        /// 单个训练样本
        /// </summary> 
        /// <param name="input"></param>
        /// <param name="output"></param>
        /// <returns></returns>
        public double Run(double[] input, double[] output)
        {
            double[] networkOutput = _network.Compute(input);
            Layer layer = _network.Layers[0];
            // 统计总误差
            double error = 0.0;

            for (int j = 0; j < layer.Neurons.Length; j++)
            {
                // 误差值，用已知的值减去网络计算出来的值
                double e = output[j] - networkOutput[j];
                // 如果存在误差，那么更新权重
                if (e != 0)
                {
                    ActivationNeuron perceptron = (ActivationNeuron)layer.Neurons[j];

                    for (int i = 0; i < perceptron.Weights.Length; i++)
                    {
                        perceptron.Weights[i] = RandRange.GetRan();
                    }

                    perceptron.Threshold = RandRange.GetRan();

                    error += Math.Abs(e);
                }
            }

            return error;
        }

        /// <summary>
        /// 训练所有样本
        /// </summary>
        /// <param name="input"></param>
        /// <param name="output"></param>
        /// <returns></returns>
        public double RunEpoch(double[][] input, double[][] output)
        {
            double error = 0.0;
            for (int i = 0, n = input.Length; i < n; i++)
            {
                error += Run(input[i], output[i]);
            }
            return error;
        }
    }
```

用随机修改参数方法要求模型很简单，本文要求的模型只是一层，也就是输入层和输出层是相同的一层。在 单个训练样本 方法将会使用模型计算出一个值，通过 `double[] networkOutput = _network.Compute(input);` 然后对比误差 `double e = output[j] - networkOutput[j];` 如果存在误差，那么用 `perceptron.Weights[i] = RandRange.GetRan();` 更新每个元的每个参数的值

现在大概写完了代码，本文的代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/ab03ad2248c0db95112edde9a4ffd7173be8f028/Bp) 下载用 VisualStudio 打开 Bp.sln 文件，然后按下 F5 就可以运行

尝试用这个模型和训练方法做出一个模拟二进制 与 的计算，也就是输入有两个，输出是件这两个输入进行 与 运算

```
0,0 = 0
0,1 = 0
1,0 = 0
1,1 = 1
```

定义输入数据

```
            double[][] inputs = new double[4][];
            double[][] outputs = new double[4][];

            //(0,0);(0,1);(1,0)
            inputs[0] = new double[] { 0, 0 };
            inputs[1] = new double[] { 0, 1 };
            inputs[2] = new double[] { 1, 0 };

            outputs[0] = new double[] { 0 };
            outputs[1] = new double[] { 0 };
            outputs[2] = new double[] { 0 };

            //(1,1)
            inputs[3] = new double[] { 1, 1 };
            outputs[3] = new double[] { 1 };
```

创建人工智能网络

```
            ActivationNetwork network = new ActivationNetwork(
                new ThresholdFunction(), 2, 1);
```

这个人工智能网络使用输入层有两个，只有一层网络，一层网络里面只有一个神经元

创建训练方法

```
            SlowLearning teacher = new SlowLearning(network);
```

然后用训练方法训练模型

```
            int iteration = 1;
            while (true)
            {
                double error = teacher.RunEpoch(inputs, outputs);
                Console.WriteLine($"迭代次数:{iteration},总体误差:{error} ");

                if (error == 0)
                    break;
                iteration++;
            }

```

尝试运行代码，可以看到我没有告诉人工智能如何做 与 运算，但是人工智能模拟了方法

尝试训练人工智能模型模拟二进制或计算

```
            double[][] inputs = new double[4][];
            double[][] outputs = new double[4][];

            //(0,0)
            inputs[0] = new double[] { 0, 0 };
            outputs[0] = new double[] { 0 };

            //(1,1);(0,1);(1,0)
            inputs[1] = new double[] { 0, 1 };
            inputs[2] = new double[] { 1, 0 };
            inputs[3] = new double[] { 1, 1 };

            outputs[1] = new double[] { 1 };
            outputs[2] = new double[] { 1 };
            outputs[3] = new double[] { 1 };


            ActivationNetwork network = new ActivationNetwork(new ThresholdFunction(), 2, 1);

            SlowLearning teacher = new SlowLearning(network);

            int iteration = 1;
            while (true)
            {
                double error = teacher.RunEpoch(inputs, outputs);
                Console.WriteLine(@"迭代次数:{0},总体误差:{1}", iteration, error);

                if (error == 0)
                    break;
                iteration++;
            }
```

我没有告诉人工智能或计算的方法，但是人工智能可以训练如何计算

这样的太简单了，其实上面的模型可以做出多分类，多分类就是将一些输入分为几类。如按照二维几何距离将数据分为几类，然后让人工智能分类

```csharp
            double[][] inputs = new double[15][];
            double[][] outputs = new double[15][];

            //(0.1,0.1);(0.2,0.3);(0.3,0.4);(0.1,0.3);(0.2,0.5)
            inputs[0] = new double[] { 0.1, 0.1 };
            inputs[1] = new double[] { 0.2, 0.3 };
            inputs[2] = new double[] { 0.3, 0.4 };
            inputs[3] = new double[] { 0.1, 0.3 };
            inputs[4] = new double[] { 0.2, 0.5 };

            outputs[0] = new double[] { 1, 0, 0 };
            outputs[1] = new double[] { 1, 0, 0 };
            outputs[2] = new double[] { 1, 0, 0 };
            outputs[3] = new double[] { 1, 0, 0 };
            outputs[4] = new double[] { 1, 0, 0 };

            //(0.1,1.0);(0.2,1.1);(0.3,0.9);(0.4,0.8);(0.2,0.9)
            inputs[5] = new double[] { 0.1, 1.0 };
            inputs[6] = new double[] { 0.2, 1.1 };
            inputs[7] = new double[] { 0.3, 0.9 };
            inputs[8] = new double[] { 0.4, 0.8 };
            inputs[9] = new double[] { 0.2, 0.9 };

            outputs[5] = new double[] { 0, 1, 0 };
            outputs[6] = new double[] { 0, 1, 0 };
            outputs[7] = new double[] { 0, 1, 0 };
            outputs[8] = new double[] { 0, 1, 0 };
            outputs[9] = new double[] { 0, 1, 0 };

            //(1.0,0.4);(0.9,0.5);(0.8,0.6);(0.9,0.4);(1.0,0.5)
            inputs[10] = new double[] { 1.0, 0.4 };
            inputs[11] = new double[] { 0.9, 0.5 };
            inputs[12] = new double[] { 0.8, 0.6 };
            inputs[13] = new double[] { 0.9, 0.4 };
            inputs[14] = new double[] { 1.0, 0.5 };

            outputs[10] = new double[] { 0, 0, 1 };
            outputs[11] = new double[] { 0, 0, 1 };
            outputs[12] = new double[] { 0, 0, 1 };
            outputs[13] = new double[] { 0, 0, 1 };
            outputs[14] = new double[] { 0, 0, 1 };
```

这个数据是[如何利用 C# 实现神经网络的感知器模型](https://www.jianshu.com/p/f48adf579a9e)用到的数据

上面的数据的输入是两个数，而输出是三个数。在输出用 0 和 1 表示属于哪个类型

本文定义一层网络，在这一层网络的输出需要三个数也就是需要三个元

```
            ActivationNetwork network = new ActivationNetwork(new ThresholdFunction(), 2, 3);
```

和上面代码一样尝试让模型学习

```
            SlowLearning teacher = new SlowLearning(network);

            int iteration = 1;

            while (true)
            {
                double error = teacher.RunEpoch(inputs, outputs);
                Console.WriteLine(@"迭代次数:{0},总体误差:{1}", iteration, error);

                if (error == 0)
                    break;
                iteration++;
            }
```

这就是一个简单的人工智能模型，所有代码都没有用到现有人工智能框架，都是使用 dotnet 基础的代码。用 C# 实现人工智能模型最成熟的是 ML.NET 但是这个库没有基础很难知道是做什么

本文的代码放在[github](https://github.com/lindexi/lindexi_gd/tree/ab03ad2248c0db95112edde9a4ffd7173be8f028/Bp) 欢迎小伙伴访问

其实人工智能的一个核心是训练算法，本文告诉大家的是 Slow 算法，这个算法就是在人工智能模型输出的值和我知道的值不同时，让模型随机更新参数。这个做法虽然能完成，但是效率很低，特别在元的数量多的时候。此时就需要用到比较高级的训练方法，如[如何利用 C# 实现神经网络的感知器模型](https://www.jianshu.com/p/f48adf579a9e)

特别感谢[老马的程序人生](https://www.jianshu.com/u/16f2e309c8ec)提供的模型

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
