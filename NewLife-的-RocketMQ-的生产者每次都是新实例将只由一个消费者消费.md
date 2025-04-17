
# NewLife 的 RocketMQ 的生产者每次都是新实例将只由一个消费者消费

我写了一点逗比代码，让在每次使用 NewLife 的 RocketMQ 发送消息时，都创建一个新的 Producer 生产者。此时我发现了在我的消费者里面，无论开多少个消费者实例进程，每次都只有一个消费者进行消费

<!--more-->


<!-- CreateTime:2021/6/16 17:19:23 -->

<!-- 发布 -->

本文记录的问题，和 NewLife 的 RocketMQ 库的设计毫无相关，仅仅只是我的逗比问题。还请大家放心使用 NewLife 的 RocketMQ 库

我在对 NewLife 的 RocketMQ 的 Producer 的逗比封装，让每次发送的时候，都不得不新建一个 Producer 实例。而有经过一些玄学的原因，如果每次的 Producer 都是新建出来的，将会导致只能有一个消费者实例去消费此消息内容

本文不去讨论玄学的原因，咱回到我的逗比代码

以下是我对 NewLife 的 RocketMQ 的 Producer 的逗比封装

```csharp
    public class RocketProducer
    {
        public string Group { get; }

        public string Topic { get; }

        public string NameServerAddress { get; }

        private NewLife.RocketMQ.Producer Producer { get; }

        public RocketProducer(RocketMQConfiguration configuration) : this(configuration.Group, configuration.Topic,
            configuration.NameServerAddress)
        {
        }

        private RocketProducer(string @group, string topic, string nameServerAddress)
        {
            Group = @group;
            Topic = topic;
            NameServerAddress = nameServerAddress;
            Producer = new NewLife.RocketMQ.Producer
            {
                NameServerAddress = NameServerAddress,
                Topic = Topic,
                Group = Group
            };
            Producer.Start();
        }

        public SendResult Send(string tag, string body)
        {
            var message = new Message
            {
                Topic = Topic,
                Keys = $"Doubi_{DateTime.Now:yyMMddHHmmssfff}_{Guid.NewGuid().ToString().Substring(0, 8)}",
                Tags = tag,
                BodyString = body
            };

            var sendResult = Producer.Publish(message);
            Producer.Dispose();
            return sendResult;
        }
    }
```

大家是否能看出来锅在哪

可以看到在每次发送完成之后，就调用了 Producer.Dispose 方法释放了生产者

因此为了使用以上逗比的封装，就需要每次都创建一个 RocketProducer 的实例去发送一条消息。简化的测试代码大概如下

```csharp
        /// <summary>
        /// 生产者
        /// </summary>
        /// 以下是逗比代码
        private void RegisterMqTaskProducerDoubi()
        {
            var group = _rocketMqConfiguration["Group"];
            var nameServerAddress = _rocketMqConfiguration["NameServerAddress"];
            var topic = _rocketMqConfiguration["Topic"];

            var rocketMqConfiguration =
                new RocketMqConfiguration(@group, nameServerAddress);

            _logger.LogInformation($"RegisterMqTaskProducer NameServerAddress={nameServerAddress};Group={group};Topic={topic}");
        
            Task.Run(async () =>
            {
                int n = 0;
                while (_foo)
                {
                    // 每次都新建一个，用来挖坑
                    var producer = new NewLife.RocketMQ.Producer
                    {
                        NameServerAddress = rocketMqConfiguration.NameServerAddress,
                        Topic = topic,
                        Group = rocketMqConfiguration.Group,
                    };

                    try
                    {
                        _logger.LogInformation("Start Producer");
                        producer.Start();
                        _logger.LogInformation("Finish Start Producer");
                    }
                    catch (Exception e)
                    {
                        _logger.LogWarning(e, e.ToString());
                        return;
                    }

                    var message = $"Message{n}";
                    _logger.LogInformation($"StartSend Topic={topic} Message={message}");
                    var result = producer.Publish(new Message()
                    {
                        Keys = $"Key{n}",
                        Topic = topic,
                        Tags = "Tag",
                        BodyString = message
                    });
                    producer.Dispose();

                    _logger.LogInformation($"FinishSend Result={result.Status};MessageId={result.MsgId};BrokerName={result.Queue.BrokerName};QueueOffset={result.QueueOffset}");

                    await Task.Delay(TimeSpan.FromSeconds(1));
                    n++;
                    if (n > 10000)
                    {
                        break;
                    }
                }
            });
        }
        private bool _foo = true;
```

大概就是每次都新建一个 Producer 用来发送

消费者的代码如下

```csharp
    class FakeConsumer
    {
        public FakeConsumer(ILogger<MqHostedService> logger, IConfigurationSection rocketMqConfiguration, string name)
        {
            _logger = logger;
            _rocketMqConfiguration = rocketMqConfiguration;
            Name = name;
        }

        /// <summary>
        /// 消费者
        /// </summary>
        public void RegisterMqTaskConsumer()
        {
            var group = _rocketMqConfiguration["Group"];
            var nameServerAddress = _rocketMqConfiguration["NameServerAddress"];
            var topic = _rocketMqConfiguration["Topic"];

            _logger.LogInformation($"RegisterMqTaskConsumer NameServerAddress={nameServerAddress};Group={group};Topic={topic}");

            var consumer = new Consumer
            {
                Group = group,
                NameServerAddress = nameServerAddress,
                Topic = topic,
            };
            consumer.BatchSize = 1; // 一次消费一个任务
            consumer.AutoSchedule = true;
            consumer.ConsumerInterval = 1000; // 消费间隔：1s

            consumer.OnConsume += OnConsume;
            consumer.Start();
        }

        private bool OnConsume(MessageQueue messageQueue, MessageExt[] messages)
        {
            try
            {
                _logger.CCloudInfo($"++{Name}+++ Message={messages[0].BodyString}",
                    string.Empty);
            }
            catch (Exception e)
            {
                _logger.CCloudInfo($"{messageQueue.QueueId} {messageQueue.BrokerName} MessageCount={messages.Length}",
                    e);
            }

            Task.Delay(TimeSpan.FromSeconds(5)).Wait();
            _logger.CCloudInfo($"--{Name}-- Message={messages[0].BodyString}",
                string.Empty);

            // 返回 true 表示这个消息被消费成功
            // 返回 false 表示这个消息消费失败，将会再次被投到消费者，但不一定再次被这个实例收到
            //return _random.Next(10) > 5;
            return true;
        }

        private Random _random = new Random();

        private readonly ILogger<MqHostedService> _logger;
        public string Name { get; }
        private IConfigurationSection _rocketMqConfiguration;
    }
```

在配置文件里面写上具体的配置，大概代码如下，请将具体的配置修改为你的消息队列服务器配置

```json
{
  "Logging": 
  {
    "LogLevel": 
    {
      "Default": "Information",
      "Microsoft": "Warning",
      "Microsoft.Hosting.Lifetime": "Information"
    }
  },
  "RocketMq": 
  {
    "Group": "DoubiLindexi",
    "NameServerAddress": "doubi-test-rockermq-1.gz.lindexi.com",
    "Topic": "Foo"
  },
  "AllowedHosts": "*"
}
```

执行此进程两次，只让单个进程调用到 RegisterMqTaskProducerDoubi 方法用来生产消息。此时可以看到只有单个进程可以收到消息，而另一个进程不能收到消息

更改生产者代码作为用一个 NewLife.RocketMQ.Producer 创建消息，如下面代码

```csharp
        /// <summary>
        /// 生产者
        /// </summary>
        private void RegisterMqTaskProducer()
        {
            var group = _rocketMqConfiguration["Group"];
            var nameServerAddress = _rocketMqConfiguration["NameServerAddress"];
            var topic = _rocketMqConfiguration["Topic"];

            var rocketMqConfiguration =
                new RocketMqConfiguration(@group, nameServerAddress);

            _logger.LogInformation($"RegisterMqTaskProducer NameServerAddress={nameServerAddress};Group={group};Topic={topic}");

            var producer = new NewLife.RocketMQ.Producer
            {
                NameServerAddress = rocketMqConfiguration.NameServerAddress,
                Topic = topic,
                Group = rocketMqConfiguration.Group,
            };

            try
            {
                _logger.LogInformation("Start Producer");
                producer.Start();
                _logger.LogInformation("Finish Start Producer");
            }
            catch (Exception e)
            {
                _logger.LogWarning(e, e.ToString());
                return;
            }

            Task.Run(async () =>
            {
                while (_foo)
                {
                	// 用来让单个进程生产消息，加入断点，然后拖到下一个语句
                    await Task.Delay(1000);
                }

                int n = 0;
                while (_foo)
                {
                    var message = $"Message{n}";
                    _logger.LogInformation($"StartSend Topic={topic} Message={message}");
                    var result = producer.Publish(new Message()
                    {
                        Keys = $"Key{n}",
                        Topic = topic,
                        Tags = "Tag",
                        BodyString = message
                    });
                    _logger.LogInformation($"FinishSend Result={result.Status};MessageId={result.MsgId};BrokerName={result.Queue.BrokerName};QueueOffset={result.QueueOffset}");

                    await Task.Delay(TimeSpan.FromSeconds(1));
                    n++;
                    if (n > 10000)
                    {
                        break;
                    }
                }
            });
        }
```

此时可以看到，多个进程都能收到消息

所以如果消息队列的消息只有被有限个消费者进行消费，请了解自己的代码，是否每次发送消息都使用独立的生产者

欢迎加入 NewLife 团队群 1600800





<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。