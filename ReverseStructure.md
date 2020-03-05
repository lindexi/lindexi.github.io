# 不使用数据结构反转栈

昨天有人问我一道题，我有一个栈，我不使用其他数据结构，不使用另一个栈，把这个栈里所有数据反转。

<!--more-->
<!-- CreateTime:2018/2/13 17:23:03 -->


<div id="toc"></div>

我一听就想说，脑残，这样会有这样的，如果可以使用另一个数组，就直接把数据弹出来，然后放进去，这样就好啦。

但是他说是笔试题，我就说什么渣公司。结果是CVTE。那么这道题还是厉害，因为需要用到递归，把递归的局部放数据，这道题出的好。因为是我们公司的。

我们用递归，用的方法是减治，因为递归就是下一次的函数做好了这一次前做好的事。

假如我有一个数据，需要使用递归，那么我就需使用完递归，输入下一个的时候，数据规模比上一次小，小多少都好，这样到最后一次就是处理很少的数据，或一个，这样就很好。

另一个就是函数处理了之前做的，然后我需要做本次的，这都是递归需要想的。

那么我们就开始想，我们有一个可以把栈反转的函数，这个函数不使用数据结构，那么我们需要递归使用，使用的时候输入比这一次少，这样就好。

开始我想的就是直接把一个局部变量存我们的栈的一个数据，递归，这样出栈完我们就使用局部压入，但是其实这样不对，因为我们这样就是让直接出栈完就压，没有变化。

那么我们想另一个方法，这个是我问了大神和我说，因为我是渣渣。

首先假如我们的数据是“12345”栈底从栈顶，我们从栈顶拿出放在我们的递归局部t，这时栈底到栈顶“1234”,我们函数可以反转栈，我们调用函数，反转，这时栈底到栈顶“4321”，我们用局部g拿栈顶，这时栈底到栈顶“432”，我们调用函数反转。栈底到栈顶“234”，我们把t压，“2345”反转，得到“5432”，把g压得到“54321”我们不使用数据结构反转栈。

这个方法一开始没有一个说是对的，其实这个方法是好的，因为我们使用我们的函数反转，下一次使用只有这一次的数据-1，也就是假如我们输入5个数据，第二次使用函数是输入4个，这样最后我们就有一个数据。

我们可以使用

```csharp
        public static void RecursionReverse(Stack<Puke> stack)
        {
            if (stack.Count == 0)
            {
                return;
            }
            Puke t = stack.Pop();

            RecursionReverse(stack);

            if (stack.Count == 0)
            {
                stack.Push(t);
            }
            else
            {
                Puke g = stack.Pop();

                RecursionReverse(stack);

                stack.Push(t);

                RecursionReverse(stack);

                stack.Push(g);
            }
        }
```

我们开始是判断我们的栈是不空，如果是我们就返回，然后我们使用t把我们栈拿出一个，然后递归，这时判断是不是t是最后一个，这样就是t出就是没有，没有的话我们就是对一个反转，反转还是他，所以我们就把t放入。放入我们就不需要继续。如果t后还有，我们就用g拿出栈的一个，反转，然后把t压，反转，压入g，结束。

```csharp
import java.util.Stack ;

public class HelloWorld{
	public static void main(String []args){
		Stack<Puke> stack=new Stack<Puke>();
		Puke puke=new Puke("1");
		stack.push(puke);

		puke=new Puke("2");

		stack.push(puke);

		puke=new Puke("3");

		stack.push(puke);

		puke=new Puke("4");

		stack.push(puke);

		puke=new Puke("5");

		stack.push(puke);

		recursionReverse(stack);

		String str="";

		while(!stack.empty()){

			str+=stack.pop().paimian;

		}

		System.out.println(str);

	}

	public static void recursionReverse(Stack<Puke> stack){
		if(stack.empty()){
			return;
		}

		Puke t=stack.pop();

		recursionReverse(stack);

		if(stack.empty()){
			stack.push(t);
		}
		else{
			Puke g=stack.pop();

			recursionReverse(stack);

			stack.push(t);

			recursionReverse(stack);

			stack.push(g);
		}
	}

	static class Puke{
		public Puke(String paimian){
			this.paimian=paimian;
		}
		public String paimian;

	}
}
```


