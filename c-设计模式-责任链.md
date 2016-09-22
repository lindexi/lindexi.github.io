#C# 设计模式 责任链

责任链模式是一种对象的行为模式。在责任链模式里，很多对象由每一个对象对其下家的引用而连接起来形成一条链。请求在这个链上传递，直到链上的某一个对象决定处理此请求。发出这个请求的客户端并不知道链上的哪一个对象最终处理这个请求，这使得系统可以在不影响客户端的情况下动态地重新组织和分配责任。《JAVA与模式》

我们在C#也可以使用责任链。

首先我们需要一个接口`IHandle`接受我们的责任，在里面，最简单的责任链只有

```
    public interface IHandle
    {
        IHandle Successor
        {
            set;
            get;
        }

        void Request(string str);

        void AddSuccessor(IHandle successor);
    }
```

责任链的下一个责任接受：Successor

处理责任：Request

添加处理责任的下一个：AddSuccessor

然后我们需要一个实际处理类，这个类集成接口`IHandle`。可以看到我的接口`IHandle `只是处理字符串，其实我们可以处理很多的，但是为了简单，我们就先写字符串。

假如我们是员工，发起的请求是叫老板加工资，那么开始决定工资还不是他，需要经过主管、HR、然后是老板，他们组成了一个责任链。

首先我们定义员工，他可以发送出责任需要让我们的具体处理者处理，但是我们这时看到了主管等其实有重复的，如果主管不同意处理，那么就没必要进行HR同意，所以我们的`IHandle`有`AreHandle`是不是被处理，因为无论是不是被处理还是要传给下一个，他需要决定这个处理我要不要继续，如果我们的员工提出要求，主管看他不爽，每次都不给他提工资，而我们HR也因为主管处理了，他就不接受，那么这样，员工就得不到提工资，为了解决，我们让HR也要接收到，也就是不管如何都把这个传到下一层，当然这样就需要传下一个`AreHandle`。如果觉得不需要，那么就不用传下。

另一个，我们不知道我们什么时候实现主管、什么时候实现HR，我们责任链的位置重要，因为如果主管处理了，HR就可以不处理，那么我们需要顺序，一般在添加那里加上权限，这里写`Access`。我们在添加具体处理，一般判断我们的下一个处理是否存在，如果不存在，直接添加输入参数下一个处理，如果存在，判断权限大小，如果比他大就代换他，如果比他小，就给下一个处理。

```
        public void AddSuccessor(IHandle successor)
        {
            if (Successor == null)
            {
                Successor = successor;
            }
            else
            {
                if (successor.Access > Successor.Access)
                {
                    var temp = Successor;
                    Successor = successor;
                    successor.Successor = temp;
                }
                else
                {
                    Successor.AddSuccessor(successor);
                }
            }
        }
```

我们发现如果把具体处理写为一个类，然后实现，这样每次使用函数`Request`都用成员，这样不太好，因为我们有功能扩展，所以我们把`ConcreteHandler`写为抽象，然后继承，写出主管等的类。

```
    public class GHandle : ConcreteHandler
    {
        public override void Request(string str)
        {
            NotifyProperty.Reminder?.Invoke("经理处理"+str);
            Successor?.Request(str);
        }
    }

    public class DeptHandle : ConcreteHandler
    {
        public override void Request(string str)
        {
            NotifyProperty.Reminder?.Invoke("主管处理"+str);
            Successor?.Request(str);           
        }
    }
```


##后退按钮使用责任链

我看到堆栈炸了有人问我，为什么一按后退就炸。

我看了他的源代码，他每个页面都把后退按钮点击事件+=他的方法。



