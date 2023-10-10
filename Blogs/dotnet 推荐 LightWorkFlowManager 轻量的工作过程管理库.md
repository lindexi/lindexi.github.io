本文将和大家推荐我团队开源的 LightWorkFlowManager 轻量的工作过程管理库，适合任何需要执行工作过程的应用逻辑，可以方便将多个工作过程拼凑起来，且自动集成重试和失败处理，以及日志和上报功能

<!--more-->


<!-- CreateTime:2023/10/8 20:19:35 -->

<!-- 发布 -->
<!-- 博客 -->

这个 LightWorkFlowManager 轻量的工作过程管理库是我所在的团队在 GitHub 上使用最友好的 MIT 协议开源的库，请看 [https://github.com/dotnet-campus/LightWorkFlowManager](https://github.com/dotnet-campus/LightWorkFlowManager)

这个 LightWorkFlowManager 轻量的工作过程管理库现在已经在我团队的正式产品应用上运行有半年了，相对来说比较稳定。使用过程中如果有任何问题，都欢迎在 GitHub 上新建 Issues 进行反馈

世界上有许多现有的工作过程管理，那为什么还重新造了一个？没啥原因，自己造的用的顺手。这个 LightWorkFlowManager 库在功能上和其他的工作过程管理库相比要缺失许多，但胜在轻量。且 LightWorkFlowManager 最重要的卖点就是对调试友好，适合用在本身复杂度就比较好的逻辑情况下，可以有效降低引入工作过程管理模块本身带来的额外复杂度

许多现有的功能齐全的工作过程管理库都是在 ASP.NET Core 上使用的。然而我这里的需求是在客户端应用框架上使用，且大部分情况下是在用户的设备上运行的，许多服务依赖本身是不存在的。尽管许多现有的功能齐全的工作过程管理库都可以通过配置的形式将其切换到单机内运行，然而这样做将在大部分时候不是库的推荐用法，用起来没有那么顺。为此我所在的团队开发了 LightWorkFlowManager 轻量的工作过程管理库，这个库可以很方便在客户端框架上运行，比如在 WPF 应用程序、在 WinForms 应用程序、在 MAUI 应用程序上运行。同时 LightWorkFlowManager 也兼顾 ASP.NET Core 服务本身，特别是依赖注入部分，可以和默认的服务的依赖注入相连接

我将举一个例子来告诉大家这个 LightWorkFlowManager 库所适用的情况。假定我拿到了一个开发任务，这个任务是做 PPT 解析，取出 PPT 文件里面的图片。细分这个任务将包括以下三个步骤：步骤一，获取 PPT 文件。步骤二，调用某个现有帮助方法获取到 PPT 文件里面的图片。步骤三，对拿到的 PPT 文件里面的图片文件进行处理

在步骤一里面，有多个方式获取到 PPT 文件，比如有些是从网上 CDN 服务器上下载的，有些是从磁盘路径获取的，有些是用户拖入的。这些不同的输入获取方式来自于各个业务的不同入口

在步骤三里面，又根据不同的业务情况，需要分为将图片文件进行上传到后台服务器，或者是有些业务需要将这些图片展示到界面上等等

从以上的描述可以看到，我所拿到的是一个可以散落在各个业务里面，看似存在许多不同，却又在逻辑上相似的需求。逻辑上的相似也就是可以强行归类步骤进行约束，逻辑上的重复在于有许多逻辑是完全可复用的，比如记录日志和重试机制。重试机制几乎可以用在以上三个步骤里面，无论是开发新手还是富有经验的开发者都知道，无论是从网上下载内容还是上传文件都需要经过不稳定的网络传输，这个时候因为网络传输失败而执行重试逻辑是十分常见的处理逻辑

假定没有 LightWorkFlowManager 库的辅助，一个可能的实现设计大概如下：设计一个统一的入口，通过入口传入参数判断执行不同的步骤处理逻辑，如下图

<!-- ![](image/dotnet 推荐 LightWorkFlowManager 轻量的工作过程管理库/dotnet 推荐 LightWorkFlowManager 轻量的工作过程管理库0.png) -->
![](http://image.acmx.xyz/lindexi%2F20231091747426125.jpg)

通过上图可以很简单就看出来问题，整个的逻辑复杂度将会比较高，原本的业务入口就存在多个，接着进入到 PPT 解析任务里面的统一入口，在 PPT 解析任务里面的统一入口再根据参数分发逻辑到不同的处理逻辑上。整个逻辑的圈复杂度将比较高，同时对于后续如果在步骤一或步骤三再添加更多的业务需求，将会让整个逻辑更加复杂起来。另一方面，如此复杂的逻辑，将会导致代码阅读和调试的困难性，比如难以非常明确的知道整个的调用链，调用链可能过于复杂，比如难以在调试的时候进行重复执行

在有 LightWorkFlowManager 库的辅助下，可以修改一下代码的逻辑设计，也就是不再进行统一的入口的分发，而是让各自的业务串联对应的逻辑，如此可以理顺一条逻辑，保持 PPT 解析任务模块的简单。同时业务方也可以更加清楚 PPT 解析任务的细节

<!-- ![](image/dotnet 推荐 LightWorkFlowManager 轻量的工作过程管理库/dotnet 推荐 LightWorkFlowManager 轻量的工作过程管理库1.png) -->
![](http://image.acmx.xyz/lindexi%2F20231091759381085.jpg)

可以看到修改之后的代码，可以让每个逻辑都是顺下来的，不存在逻辑的圈。可以将原本就比较复杂的逻辑，降低其逻辑复杂度。一个串联的顺的逻辑，将对代码阅读以及调试更加友好。且在 LightWorkFlowManager 库的辅助之下，可以尽量减少重复的代码逻辑，比如减少重复的记录日志和失败重试逻辑

在了解 LightWorkFlowManager 库的适用设计之后，接下来将和大家介绍具体的使用方法

按照 dotnet 的惯例，先安装 NuGet 库，请看 [https://www.nuget.org/packages/dotnetCampus.LightWorkFlowManager](https://www.nuget.org/packages/dotnetCampus.LightWorkFlowManager)

在开始介绍使用方法之前，需要先介绍一下 LightWorkFlowManager 里面的概念。在 LightWorkFlowManager 里面定义了 MessageWorkerManager 类型，将其作为工作器的管理器，用途就是将各个工作器串联起来，且运行起来，这个 MessageWorkerManager 就是 LightWorkFlowManager 的入口类型

在 LightWorkFlowManager 里面定义了工作器的基础类型就是 MessageWorker 类型，所有的业务上的开发者自定义的工作器都需要直接或间接继承 MessageWorker 类型

了解基础概念之后，接下来就开始编写一个简单的逻辑作为例子

## 使用方法

1、 创建 MessageWorkerManager 对象。创建 MessageWorkerManager 工作器管理器即可作为承载工作器的框架，请为每次单独的任务创建独立的 MessageWorkerManager 对象

```csharp
            // 每个任务一个 TaskId 号
            string taskId = Guid.NewGuid().ToString();
            // 相同类型的任务采用相同的名字，比如这是做 PPT 解析的任务
            string taskName = "PPT 解析";
            // 提供容器
            IServiceScope serviceScope = serviceProvider.CreateScope();

            var workerManager = new MessageWorkerManager(taskId, taskName, serviceScope);
```

2、 定义 Worker 工作器。以下例子代码定义了模拟业务代码的 FooWorker 工作器，可以在 FooWorker 重写的 DoInnerAsync 方法里面编写执行的业务代码。以下代码的 InputType 和 OutputType 分别是工作器的输入和输出类型，每个工作器的输出对象都会自动加入到 MessageWorkerManager 的过程上下文缓存里面

```csharp
record InputType();

record OutputType();

class FooWorker : MessageWorker<InputType, OutputType>
{
    protected override ValueTask<WorkerResult<OutputType>> DoInnerAsync(InputType input)
    {
        ...
    }
}
```

3、 执行 Worker 工作器。 完成类型的定义之后，接下来演示如何通过 MessageWorkerManager 将 FooWorker 运行起来

```csharp
            var result = await workerManager
                .GetWorker<FooWorker>()
                .RunAsync();
```

以上代码也可以简写为以下代码

```csharp
            var result = await workerManager.RunWorker<FooWorker>();
```

以上代码运行的前提是先注入 FooWorker 所需的 InputType 类型的对象。下文将告诉注入工作器参数部分的更多细节

## 机制和功能

### 工作器参数

工作器参数通过 MessageWorkerManager 工作器管理的 IWorkerContext 上下文读取。每个工作器的可返回值类型都会自动被设置到 IWorkerContext 上下文里面。如此即可自动实现上一个 Worker 的输出作为下一个 Worker 的输入

在每个工作器里面，都可以通过 `SetContext` 设置上下文信息

在开始执行工作器时，还可以手动设置输入参数，如以下例子

```csharp
 // 例子1：先获取工作器，再赋值给到工作器的执行方法

            await Manager
                .GetWorker<FooWorker>()
                .RunAsync(new InputType());

 // 例子2： 通过 SetContext 进行设置参数，再执行工作器
            await Manager
                .SetContext(new InputType())
                .RunWorker<FooWorker>();
```

如果有些工作器之间的输入和输出参数需要进行转换，也可以 `SetContext` 传入转换委托进行参数转换

```csharp
 // 以下例子将当前上下文里的 Foo1Type 类型转换为 FooWorker 需要的 Foo2Type 参数
           await Manager
                .SetContext((Foo1Type foo1) => ConvertFoo1ToFoo2(foo1))
                .RunWorker<FooWorker>();
```

### 异常中断和重试

每个 Worker 都可以返回 WorkerResult 类型的返回值，可以在返回值里面告知框架层是否当前的 Worker 执行成功。在执行失败时，可以赋值错误码，方便定位调试或输出。在执行失败时，可以返回告知框架层是否需要重试，默认情况下框架都会自动进行重试

中断后续的工作器执行有两个方法：

方法1： 通过返回状态为失败的 WorkerResult 返回值。一旦工作管理器的状态为 IsFail 状态，将会阻止所有的没有标记 CanRunWhenFail 为 true 的工作器的执行。换句话说就是除了哪些不断成功或失败状态都要执行的 Worker 工作器之外，其他的工作器都将不会执行，包括 SetContext 里面的委托转换也不会执行

方法2： 通过抛出异常的方式，通过 dotnet 里面的异常可以让后续逻辑炸掉不再执行

以上两个方法都是框架推荐使用的。框架设计的偏好是如果在重视性能的情况下，尽量使用方法1的方式中断执行。如果是在复杂的业务逻辑，有大量业务逻辑穿插在工作过程之外，则可以方便通过方法2进行中断

### 在 Worker 里执行其他 Worker 工作器

在一个 Worker 里面，可以执行其他的 Worker 工作器，如此可以比较自由的实现分支逻辑，套娃决定执行工作器

例子如下：

假定有一个 Worker2 工作器，定义如下：

```csharp
    class Worker2 : MessageWorker<InputType, OutputType>
    {
        protected override ValueTask<WorkerResult<OutputType>> DoInnerAsync(InputType input)
        {
            return SuccessTask(new OutputType());
        }
    }

    record InputType();

    record OutputType();
```

有另一个 Worker1 工作器，可以在 Worker1 里面执行 Worker2 工作器：

```csharp
    class Worker1 : MessageWorkerBase
    {
        public override async ValueTask<WorkerResult> Do(IWorkerContext context)
        {
            await Manager
                .GetWorker<Worker2>()
                .RunAsync(new InputType());

            return WorkerResult.Success();
        }
    }
```

### 委托工作器

有一些非常小且轻的逻辑，也想加入到工作过程里面，但不想为此定义一个单独的工作器。可以试试委托工作器，如以下代码例子

```csharp
            var delegateMessageWorker = new DelegateMessageWorker(_ =>
            {
                // 在这里编写委托工作器的业务内容
            });

            var result = await messageWorkerManager.RunWorker(delegateMessageWorker);
```

如果连 DelegateMessageWorker 都不想创建，那也可以直接使用 MessageWorkerManager 的 RunWorker 方法传入委托，如以下代码例子

```csharp
            await messageWorkerManager.RunWorker((IWorkerContext context) =>
            {
                // 在这里编写委托工作器的业务内容
            });
```
