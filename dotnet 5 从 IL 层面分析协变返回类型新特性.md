# dotnet 5 从 IL 层面分析协变返回类型新特性

在 C# 9.0 里面添加的一个新特性是支持协变返回类型，也就说子类重写了基类的抽象或虚拟方法，可以在返回值里面返回协变的类型，也就是返回值的类型可以是继承原本子类返回值类型的子类。本文将来从 IL 的层面和运行时告诉大家这个新特性为什么需要 dotnet 5.0 才能支持

<!--more-->
<!-- CreateTime:2021/3/9 19:10:07 -->

<!-- 发布 -->

在开始之前，必须说明的是 C# 语言和 .NET 框架是分开的，不能因为 C# 9.0 用到了某些只有在 dotnet 5.0 的运行时才能提供的功能就说 C# 和 .NET 绑定。实际上在 .NET Framework 4.5 都能使用大量的 C# 9.0 语法。准确来说是 C# 9.0 语法里面的有一些新的特性需要在新的运行时和框架下才能使用起来，此部分新特性将需要 .NET 5.0 的支持，其他的部分只需要编译器支持就可以，依然可以在旧版本的 .NET 运行

本文依然是底层知识，本文内容不适合新手阅读。如果不想了解底层的原理，那么只需要知道这个新特性需要 IL 的支持，因为生成的 IL 代码语法上和之前的相同，但 IL 代码逻辑和之前不兼容。因为 IL 逻辑的变更，自然也需要 CLR 运行时的特别支持。这个新特性需要 IL 和运行时的支持，在旧版本的 .NET 是不能使用的

在开始之前，大家看一下新的语法的写法。如以下代码，从 Animal 继承的 Tiger 类重写了 GetFood 方法，但是在 Tiger 的 GetFood 方法的方法返回值和 Animal 的 GetFood 方法定义的不相同

```csharp
    abstract class Animal
    {
        public virtual Food GetFood()
        {
            return null;
        }
    }

    class Tiger : Animal
    {
        public override Meat GetFood() => new Meat();
    }
```

或者说是如下写法，在 Animal 类的 GetFood 是抽象的方法

```csharp
    abstract class Animal
    {
        public abstract Food GetFood();
    }

    class Tiger : Animal
    {
        public override Meat GetFood() => new Meat();
    }
```

上面两个代码的不同在于 Animal 类使用的是 abstract 或 virtual 的方法被重写，在重写的时候可以返回协变的类。以下是返回值 Food 类型定义

```csharp
    public class Food
    {

    }

    public class Meat : Food
    {

    }
```

可以看到 Meat 是继承 Food 的类型，也就是说允许子类的返回值类型是重写的方法的子类。这是一个不错的特性，可惜在 .NET Framework 下是用不了的，因为需要 CLR 运行时和框架的支持

上面开源，可以在 [github](https://github.com/lindexi/lindexi_gd/tree/99f55216/BairbacearbakurgaicairCearlellerfenall ) 或 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/99f55216/BairbacearbakurgaicairCearlellerfenall ) 下载全部代码

先从 IL 的层面来聊聊这个新特性的不同，如下面的 C# 代码，这是不使用新特性的方法

```csharp
    abstract class Animal
    {
        public abstract Food GetFood();
    }

    class Tiger : Animal
    {
        public override Food GetFood() => new Meat();
    }
```

上面代码生成的 IL 代码大概如下

```IL
  .method public hidebysig virtual instance class Lindexi.Food
    GetFood() cil managed
  {
    .maxstack 8

    // [20 43 - 20 53]
    IL_0000: newobj       instance void Lindexi.Meat::.ctor()
    IL_0005: ret

  } // end of method Tiger::GetFood
```

上面的 IL 代码咱现在还不需要去阅读他，接下来生成一下使用新特性的如以下 C# 代码的 IL 代码

```csharp
    abstract class Animal
    {
        public abstract Food GetFood();
    }

    class Tiger : Animal
    {
        public override Meat GetFood() => new Meat();
    }
```

上面 C# 生成的 IL 代码如下

```IL
  .method public hidebysig virtual newslot instance class Lindexi.Meat
    GetFood() cil managed
  {
    .custom instance void [System.Runtime]System.Runtime.CompilerServices.PreserveBaseOverridesAttribute::.ctor()
      = (01 00 00 00 )
    .override method instance class Lindexi.Food Lindexi.Animal::GetFood()
    .maxstack 8

    // [20 43 - 20 53]
    IL_0000: newobj       instance void Lindexi.Meat::.ctor()
    IL_0005: ret

  } // end of method Tiger::GetFood
```

对比和没有使用新特性的 IL 代码，从方法的定义上，就可以看到一些不同点，下面是两个 IL 的对比

```IL
.method public hidebysig virtual         instance class Lindexi.Food GetFood() cil managed
.method public hidebysig virtual newslot instance class Lindexi.Meat GetFood() cil managed
```

可以看到使用新特性的 IL 代码多了 `newslot` 关键字，这个 IL 关键词其实就相当于使用 `new` 关键字进行重写子类的方法，可以认为和子类的方法是两个不同的方法。但实际上又是在做继承方法，在 IL 的设计里面，为了让方法返回值不相同，此时就使用 `newslot` 关键字表示这是一个新的独立的方法，但又不能让这个方法和原本的代码逻辑不同，因此又需要让这个子类方法继承基类方法，于是就再加上了以下两行代码

```IL
    .custom instance void [System.Runtime]System.Runtime.CompilerServices.PreserveBaseOverridesAttribute::.ctor() = (01 00 00 00 )
    .override method instance class Lindexi.Food Lindexi.Animal::GetFood()
```

上面 IL 的第一句话是添加了 PreserveBaseOverridesAttribute 这个特性，也就是在 Roslyn 生成 IL 逻辑自动给这个函数加上了 PreserveBaseOverridesAttribute 特性，相当于以下代码

```csharp
    class Tiger : Animal
    {
        [PreserveBaseOverridesAttribute]
        public override Meat GetFood() => new Meat();
    }
```

上面的 PreserveBaseOverridesAttribute 特性是自动添加的，不需要手动加上。在这个方法的下一句是 `.override method` 表示实际这个方法是有继承某个方法的，代码如下

```IL
    .override method instance class Lindexi.Food Lindexi.Animal::GetFood()
```

通过上面的 IL 代码就可以在 CLR 找到重写的方法

上面代码的 PreserveBaseOverridesAttribute 特性是 .NET 5 框架提供的类型，也就是说 .NET Framework 4.8 等是没有这个类的

接着从 CLR 层面来讲这个新特性，如上面 IL 代码，和原本的 IL 不是兼容的，需要 CLR 层面做一些逻辑才能了解上面的 IL 的逻辑含义。需要说明的是，上面 IL 的语法含义依然是兼容的，但是逻辑含义不是兼容的，需要运行时做一些逻辑才能了解这个 IL 代码表示 GetFood 方法继承的方法

在 `src\coreclr\vm\class.cpp` 的 `ClassLoader::PropagateCovariantReturnMethodImplSlots` 方法里面是处理这个新特性的核心逻辑，在 PropagateCovariantReturnMethodImplSlots 方法会先判断是否存在 PreserveBaseOverridesAttribute 特性，如果存在那么继续通过 IL 里面记录的 `.override method` 找到实际的关系，代码如下

```c++
void ClassLoader::PropagateCovariantReturnMethodImplSlots(MethodTable* pMT)
{
    CONTRACTL
    {
        STANDARD_VM_CHECK;
        PRECONDITION(CheckPointer(pMT));
    }
    CONTRACTL_END;

    // 以下代码的逻辑是如果 MethodImpl 具有 PreserveBaseOverridesAttribute 特性，则将重写的 MethodImpl 传播到所有适用的虚表空间槽。在 C# 的抽象或虚拟方法都相当于定义了函数的虚表，存放在虚表空间槽。 这是为了确保如果我们使用基类型方法之一的签名来调用覆盖方法，我们仍然执行覆盖方法。
    // 例如下面注释的代码例子
    //
    // Propagate an overriding MethodImpl to all applicable vtable slots if the MethodImpl
    // has the PreserveBaseOverrides attribute. This is to ensure that if we use the signature of one of
    // the base type methods to call the overriding method, we still execute the overriding method.
    //
    // Consider this case:
    //
    //      class A 
    //      {
    //          RetType VirtualFunction() { }
    //      }
    //      class B : A 
    //      {
    //          [PreserveBaseOverrides]
    //          DerivedRetType VirtualFunction() { .override A.VirtualFuncion }
    //      }
    //      class C : B 
    //      {
    //          MoreDerivedRetType VirtualFunction() { .override A.VirtualFunction }
    //      }
    //
    // NOTE: Typically the attribute would be added to the MethodImpl on C, but was omitted in this example to
    //       illustrate how its presence on a MethodImpl on the base type can propagate as well. In other words,
    //       think of it as applying to the vtable slot itself, so any MethodImpl that overrides this slot on a
    //       derived type will propagate to all other applicable vtable slots.
    //
    // Given an object of type C, the attribute will ensure that:
    //      callvirt RetType A::VirtualFunc()               -> executes the MethodImpl on C
    //      callvirt DerivedRetType B::VirtualFunc()        -> executes the MethodImpl on C
    //      callvirt MoreDerivedRetType C::VirtualFunc()    -> executes the MethodImpl on C
    //
    // Without the attribute, the second callvirt would normally execute the MethodImpl on B (the MethodImpl on
    // C does not override the vtable slot of B's MethodImpl, but only overrides the declaring method's vtable slot.
    //

    // Validation not applicable to interface types and value types, since these are not currently
    // supported with the covariant return feature

    if (pMT->IsInterface() || pMT->IsValueType())
        return;

    MethodTable* pParentMT = pMT->GetParentMethodTable();
    if (pParentMT == NULL)
        return;

    // Propagate overriding MethodImpls to applicable vtable slots if the declaring method has the attribute

    if (pMT->GetClass()->HasVTableMethodImpl())
    {
        MethodTable::MethodDataWrapper hMTData(MethodTable::GetMethodData(pMT, FALSE));

        for (WORD i = 0; i < pParentMT->GetNumVirtuals(); i++)
        {
            if (pMT->GetRestoredSlot(i) == pParentMT->GetRestoredSlot(i))
            {
                // The real check is that the MethodDesc's must not match, but a simple VTable check will
                // work most of the time, and is far faster than the GetMethodDescForSlot method.
                _ASSERTE(pMT->GetMethodDescForSlot(i) == pParentMT->GetMethodDescForSlot(i));
                continue;
            }

            MethodDesc* pMD = pMT->GetMethodDescForSlot(i);
            MethodDesc* pParentMD = pParentMT->GetMethodDescForSlot(i);
            if (pMD == pParentMD)
                continue;

            // If the bit is not set on this method, but we reach here because it's been set on the method at the same slot on
            // the base type, set the bit for the current method to ensure any future overriding method down the chain gets checked.
            if (!pMD->RequiresCovariantReturnTypeChecking() && pParentMD->RequiresCovariantReturnTypeChecking())
                pMD->SetRequiresCovariantReturnTypeChecking();

            // The attribute is only applicable to MethodImpls. For anything else, it will be treated as a no-op
            if (!pMD->IsMethodImpl())
                continue;

            // Search if the attribute has been applied on this vtable slot, either by the current MethodImpl, or by a previous
            // MethodImpl somewhere in the base type hierarchy.
            bool foundAttribute = false;
            MethodTable* pCurrentMT = pMT;
            while (!foundAttribute && pCurrentMT != NULL && i < pCurrentMT->GetNumVirtuals())
            {
                MethodDesc* pCurrentMD = pCurrentMT->GetMethodDescForSlot(i);

                // The attribute is only applicable to MethodImpls. For anything else, it will be treated as a no-op
                if (pCurrentMD->IsMethodImpl())
                {
                	// 下面两个变量是没有使用的，只是让 GetCustomAttribute 函数可以调
                    BYTE* pVal = NULL;
                    ULONG cbVal = 0;
                    // 这里就是判断是否存在特性，如果存在，那么设置 foundAttribute 变量
                    if (pCurrentMD->GetCustomAttribute(WellKnownAttribute::PreserveBaseOverridesAttribute, (const void**)&pVal, &cbVal) == S_OK)
                        foundAttribute = true;
                }

                pCurrentMT = pCurrentMT->GetParentMethodTable();
            }

            if (!foundAttribute)
                continue;

            // Search for any vtable slot still pointing at the parent method, and update it with the current overriding method
            for (WORD j = i; j < pParentMT->GetNumVirtuals(); j++)
            {
                MethodDesc* pCurrentMD = pMT->GetMethodDescForSlot(j);
                if (pCurrentMD == pParentMD)
                {
                    // This is a vtable slot that needs to be updated to the new overriding method because of the
                    // presence of the attribute.
                    pMT->SetSlot(j, pMT->GetSlot(i));
                    _ASSERT(pMT->GetMethodDescForSlot(j) == pMD);

                    hMTData->UpdateImplMethodDesc(pMD, j);
                }
            }
        }
    }
}
```

上面代码在 dotnet 的运行时开源仓库里面，请看 [https://github.com/dotnet/runtime/](https://github.com/dotnet/runtime/) 源代码

在 Mono 里面，当前的 Mono 也是放在 [https://github.com/dotnet/runtime/](https://github.com/dotnet/runtime/) 里面，也对这个新特性做了自己的实现，在 Mono 的 `src\mono\mono\metadata\class-init.c` 里面将会使用如下代码判断某个方法是否有 PreserveBaseOverridesAttribute 特性

```c
gboolean
mono_class_setup_method_has_preserve_base_overrides_attribute (MonoMethod *method)
{
	MonoImage *image = m_class_get_image (method->klass);
	/* FIXME: implement well known attribute check for dynamic images */
	if (image_is_dynamic (image))
		return FALSE;
	return method_has_wellknown_attribute (method, "System.Runtime.CompilerServices", "PreserveBaseOverridesAttribute", TRUE);
}
```

如果判断存在 PreserveBaseOverridesAttribute 也就在 `src\mono\mono\metadata\class-setup-vtable.c` 的 `mono_class_setup_vtable_general` 方法里面进行后续的逻辑，因为 mono_class_setup_vtable_general 方法太长了，而我对 Mono 的实现也不熟悉，更多细节还请大家阅读源代码

特别感谢 [少珺](https://blog.sdlsj.net/) 小伙伴给我的协助

文档请看

- [Covariant return types - C# 9.0 specification proposals](https://docs.microsoft.com/en-us/dotnet/csharp/language-reference/proposals/csharp-9.0/covariant-returns?WT.mc_id=WD-MVP-5003260 )
- [C# 9.0 中的新增功能 - C# 指南](https://docs.microsoft.com/zh-cn/dotnet/csharp/whats-new/csharp-9?WT.mc_id=WD-MVP-5003260 )
- [继承 - C# 编程指南](https://docs.microsoft.com/zh-cn/dotnet/csharp/programming-guide/classes-and-structs/inheritance?WT.mc_id=WD-MVP-5003260 )

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://i.creativecommons.org/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
