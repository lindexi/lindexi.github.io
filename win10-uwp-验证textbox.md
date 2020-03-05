# win10 uwp 验证输入 自定义用户控件

TextBox是给用户输入，我们有时要用户只输入数字，而用户输入汉字，我们就有提示用户，那么这东西用到次数很多，我们需要做成一个控件。

我们可以用别人的库，我找到一个[大神](http://jamescroft.co.uk)写的库，很好用

我们使用这个库可以定义很多验证，我记录我如何使用他这个库，还有如何去修改这个库。如何自定义控件做一个和大神做的一样的控件。

<!--more-->
<!-- CreateTime:2018/8/10 19:16:51 -->


<div id="toc"></div>

## Nuget

我们用这个库是jamescroft写的，他上传到Nuget，我们可以去下载

用Nuget搜索`WinUX.UWP.Xaml.Controls`

![](http://image.acmx.xyz/fc7733af-8526-44d2-84b9-99b41ef99f4a2016121214411.jpg)

下载完成就好

## 使用库

我们经常需要验证用户输入，不是使用一个规则，是有很多规则。我们常用的规则也就几个，数字、URL……

我们下载库，有常用规则

– DateTimeValidationRule 
  日期规则，输入可以转日期

– DecimalValidationRule    
  数字，输入可以转数字

– DoubleValidationRule
  输入可以转浮点

– EmailValidationRule
  邮箱，输入是mail

– IntValidationRule
  整形，输入可以转整形

– RegexValidationRule
  我们自己定义正则，有时我们需要复杂的，要求长度

– UrlValidationRule
  输入是URL

我们要在资源定义，因为我们有多条验证

先写，因为我们引用库和我的不知一空间
		
		

```csharp
     xmlns:validation="using:WinUX.Data.Validation"
     xmlns:rules="using:WinUX.Data.Validation.Rules"
     xmlns:controls="using:WinUX.Xaml.Controls"

```

 在资源定义，我们需要多条，看起来好长，如果我们要用两次，那么还是写资源


```xml
        <Grid.Resources>
            <validation:ValidationRules x:Key="UrlSample">
                <validation:ValidationRules.Rules>
                    <rules:UrlValidationRule />
                </validation:ValidationRules.Rules>
            </validation:ValidationRules>
        </Grid.Resources>
 
```

 下面直接抄大神写的


```xml
     <controls:ValidatingTextBox IsMandatory="True" Header="Website" Text="http://www.jamescroft.co.uk" 
                                    MaxLength="50" 
                                    ValidationRules="{StaticResource Url}" 
                                    VerticalAlignment="Center"/>
 
```

![](http://image.acmx.xyz/validaTextbox20161212145937.gif)

`MaxLength` 最大长度

`ValidationRules` 验证规则

<!-- `IsMandatory` 输入是否对

这个值绑定到ViewModel可以得到是否可以输入到ViewModel -->

`IsMandatory` 输入规则

`MandatoryValidationMessage` 输入规则提示

如果`IsMandatory=true`在没有输入，显示`MandatoryValidationMessage`

`IsInvalid` 输入是否对

这个值绑定到ViewModel可以得到是否可以输入到ViewModel

如果我们需要写输入错了提示
		

```xml
     <rules:UrlValidationRule ErrorMessage="输入错"></rules:UrlValidationRule>
```

如果需要使用正则，我们的验证复制，需要使用`RegexValidationRule`，在`Regex`写正则

		

```xml
        <controls:ValidatingTextBox IsMandatory="True" Header="Website" Text="" 
                                    MaxLength="0" MandatoryValidationMessage="输入" 
                                    VerticalAlignment="Center">
            <controls:ValidatingTextBox.ValidationRules>
               <validation:ValidationRules>
                   <validation:ValidationRules.Rules>
                        <rules:RegexValidationRule Regex="mailto:\w+([-+.]\w+)*@\w+([-.]\w+)*.\w+([-.]\w+)*"></rules:RegexValidationRule>
                   </validation:ValidationRules.Rules>
               </validation:ValidationRules>
            </controls:ValidatingTextBox.ValidationRules>
            <controls:ValidatingTextBox.ValidationTextBlock >
                <TextBlock Text="ValidationTextBlock"></TextBlock>
            </controls:ValidatingTextBox.ValidationTextBlock>
        </controls:ValidatingTextBox>


```

使用很简单，那么这如何做。

下面我来说下，他这个如何做，这有些复杂，我们分开来说，开始是功能

## 判断输入字符长度

我们需要一个TextBlock来显示最大长度、现在字符长度

我们的TextBlock的名称`remainingCharacters`，在输入变化修改

		

```csharp
        private void UpdateRemainingCharacters()
        {
            //判断不为空
            if (this.remainingCharacters != null)
            {
                //如果最大长度==0，不显示
                if (this.MaxLength == 0)
                {
                    this.remainingCharacters.Visibility = Visibility.Collapsed;
                    return;
                }

                //Text是现在字符，我们用一个新字符串来得到
                var remainingChar = string.Format("{0}/{1}", this.Text.Length, this.MaxLength);

                this.remainingCharacters.Text = remainingChar;

                //这句是没用
                this.remainingCharacters.Visibility = this.MaxLength > 0 ? Visibility.Visible : Visibility.Collapsed;
            }
        }

```

上面是大神写的，我建议可以简单一点。

		

```csharp
        private void UpdateRemainingCharacters()
        {
            if (remainingCharacters != null)
            {
                if (MaxLength == 0)
                {
                    remainingCharacters.Visibility = Visibility.Collapsed;
                    return;
                }
                int length = 0;
                if (!string.IsNullOrEmpty(Text))
                {
                    length = Text.Length;
                }
                var remainingChar = string.Format("{0}/{1}", length, MaxLength);

                remainingCharacters.Text = remainingChar;

                //The above has judge the MaxLength ,and if it's 0,the Visibility will Collapsed
                //上面代码有判断MaxLength，如果是0，隐藏
                //this.remainingCharacters.Visibility = this.MaxLength > 0 ? Visibility.Visible : Visibility.Collapsed;
            }
        }

```

## 是否要检查

我们先判断是否要检查，如果不要检查，那么就返回对


```csharp
return !IsMandatory;

```

如果要检查，我们的输入是空，我们要提示用户输入

```csharp
            if (!this.IsMandatory && string.IsNullOrWhiteSpace(this.Text))
            {
                ValidationTextBlock.Text = this.MandatoryValidationMessage;
            }

```

上面代码的 ValidationTextBlock 就是提示

我们判断，如果输入不是空，就返回规则判断。

于是我们改为 如果不检查或输入是不空的，返回true。如果输入不是为空，继续使用规则代码


```csharp
ValidationTextBlock.Text = this.MandatoryValidationMessage;

```

总的代码请看：
		

```csharp
        public bool IsMandatoryFieldValid()
        {
            //不检查 || 输入不是空
            //如果不检查，true
            //如果检查 如果输入不是空 true
            if (!this.IsMandatory || !string.IsNullOrWhiteSpace(this.Text))
            {
                return true;
            }

            if (this.ValidationTextBlock != null)
            {
                this.ValidationTextBlock.Text = this.MandatoryValidationMessage;
            }

            VisualStateManager.GoToState(this, "Mandatory", true);
            return false;
        }

```


## 长度

我们通过检查验证，我们继续判断，这时我们可以检查长度 `Text.Length > this.MaxLength` ，如果大于长度，不通过，提示用户。



```csharp
            bool[] isInvalid = { !this.IsMandatoryFieldValid() };

            if (!isInvalid[0])
            {
                if (this.MaxLength > 0)
                {
                    isInvalid[0] = this.Text.Length > this.MaxLength;
                }

                if (isInvalid[0])
                {
                    if (this.ValidationTextBlock != null)
                    {
                        this.ValidationTextBlock.Text = "The text exceeds the maximum length.";
                    }
                }
            }    //不知道isInvalid 

```

## 判断

如果输入长度不大于最大可以接受的输入，我们判断是否符合要求，这样写是因为判断长度的速度比后面的判断快


```csharp
                    if (ValidationRules != null)
                    {
                        foreach (var temp in ValidationRules.Rules)
                        {
                            isInvalid[0] = !temp.IsValid(Text);
                            if (isInvalid[0] && ValidationTextBlock != null)
                            {
                                ValidationTextBlock.Text = temp.ErrorMessage;
                                break;
                            }
                        }
                    }

```

大神写的使用TakeWhile，这函数判断所有的值符合条件。


```csharp
           if (!isInvalid[0])
            {
                if (this.MaxLength > 0)
                {
                    isInvalid[0] = this.Text.Length > this.MaxLength;
                }

                if (isInvalid[0])
                {
                    if (this.ValidationTextBlock != null)
                    {
                        this.ValidationTextBlock.Text = "The text exceeds the maximum length.";
                    }
                }
                else
                {
                    if (this.ValidationRules != null)
                    {
                        // Run through all of the validation rules for this text box and check is valid.
                        foreach (var rule in this.ValidationRules.Rules.TakeWhile(rule => !isInvalid[0]))
                        {
                            isInvalid[0] = !rule.IsValid(this.Text);

                            if (isInvalid[0] && this.ValidationTextBlock != null)
                            {
                                this.ValidationTextBlock.Text = rule.ErrorMessage;
                            }
                        }
                    }

                    //if (ValidationRules != null)
                    //{
                    //    foreach (var temp in ValidationRules.Rules)
                    //    {
                    //        isInvalid[0] = !temp.IsValid(Text);
                    //        if (isInvalid[0] && ValidationTextBlock != null)
                    //        {
                    //            ValidationTextBlock.Text = temp.ErrorMessage;
                    //            break;
                    //        }
                    //    }
                    //}
                }
            }

```

我们需要把判断放到IsInvalid，`IsInvalid = isInvalid[0];`

我们把上面写的做函数，输入改变我们使用更新来做判断。因为这个函数是所有的输入都调用，所以可能规则比较慢就会让用户难以输入。

```csharp
        private void OnTextChanged(object sender, TextChangedEventArgs args)
        {
            this.Update();
        }

```

## 如何写检查

我们的核心就是它，我们需要一个类来放用户写的检查

这类我就放`public List<ValidationRule> Rules { get; private set; }`

开始核心ValidationRule，我们有很多检查，我们需要一个ValidationRule，定义的检查都可以修改ValidationRule新检查

ValidationRule只有一个属性，错误显示的Message

		

```csharp
        private string _errorMessage;

        public string ErrorMessage
        {
            set
            {
                this._errorMessage = value;
            }
            get
            {
                return string.IsNullOrWhiteSpace(this._errorMessage) ? "Field invalid." : this._errorMessage;
            }
        }
```

其实如果`_errorMessage`不存在，我们要返回默认的，不要返回"Field invalid."

		

```csharp
        /// <summary>
        /// If the errorMessage is null  
        /// return DefaultErrorMessage
        /// </summary>
        public static string DefaultErrorMessage
        {
            set;
            get;
        } = "Field invalid.";

        private string _errorMessage;

        /// <summary>
        /// Gets the error message to display for the rule.
        /// </summary>
        public string ErrorMessage
        {
            set
            {
                this._errorMessage = value;
            }
            get
            {
                return string.IsNullOrWhiteSpace(this._errorMessage) ? DefaultErrorMessage : this._errorMessage;
            }
        }

```

然后就是一个函数，判断是否通过


```csharp
public abstract bool IsValid(object value);

```

然后我们可以开始做检查

如 DateTime 的判断


```csharp
        public override bool IsValid(object value)
        {
            if (value == null)
            {
                return false;
            }

            var val = value.ToString();
            if (string.IsNullOrWhiteSpace(val))
            {
                return true;
            }

            DateTime temp;
            return DateTime.TryParse(val, out temp);
        }

```

还有数字 decimal 判断



```csharp
        public override bool IsValid(object value)
        {
            if (value == null)
            {
                return false;
            }

            var val = value.ToString();
            if (string.IsNullOrWhiteSpace(val))
            {
                return true;
            }

            decimal temp;
            return decimal.TryParse(val, out temp);
        }

```

## 用户控件 

<!-- 不知Customer Control叫用户控件？不想查，大家知道我说什么就好 -->

我们使用继承TextBox做自己的控件
		

```csharp
public partial class ValidatingTextBox : TextBox

```

我们上面用了`remainingCharacters` `ValidationTextBlock` 我们需要把它显示

我们告诉后来写 ControlTemplate 我们要`remainingCharactersTextBlock` `ValidationTextBlock`这两个，我们给 xaml 写的资源名字 RemainingCharacters，ValidationText ，我们就可以在 OnApplyTemplate 获得这两个属性
		

```csharp
        protected override void OnApplyTemplate()
        {
            base.OnApplyTemplate();

            this.remainingCharacters = this.GetTemplateChild("RemainingCharacters") as TextBlock;
            this.ValidationTextBlock = this.GetTemplateChild("ValidationText") as TextBlock;
        }    

```

但我们需在ValidatingTextBox 的类上面写下面的代码，告诉 xaml 需要有这两个属性，虽然不写也是不会报错的，但是一般都会写。	

```csharp
    [TemplatePart(Name = "ValidationText", Type = typeof(TextBlock))]
    [TemplatePart(Name = "RemainingCharacters", Type = typeof(TextBlock))]

```

垃圾wr做这是是支持做界面的人和做逻辑可以两个。，做界面只要知道有那些控件就好
TemplatePart 是告诉做界面，我的需要名字为 Name，类型为什么的控件，你要做前台写这个控件。

写完了界面，我们还需加事件，获得文字修改
		

```csharp
       protected override void OnApplyTemplate()
        {
            base.OnApplyTemplate();

            this.remainingCharacters = this.GetTemplateChild("RemainingCharacters") as TextBlock;
            this.ValidationTextBlock = this.GetTemplateChild("ValidationText") as TextBlock;

            this.Update();

            if (!this.IsTemplateApplied)
            {
                this.TextChanged += this.OnTextChanged;
                this.IsEnabledChanged += this.OnIsEnabledChanged;

                this.IsTemplateApplied = true;
            }
        }

```

我们设Style，他没有Key，这样所有的控件都使用我们写的 Style

我们新建一个资源，只要里面有`<Style TargetType="controls:ValidatingTextBox">`

我们用新建副本，直接复制TextBox的Style，不需要做什么修改。

我们在下面，修改显示

我们需要一个Head、一个显示字符数、一个验证，TextBlock

但是还记得我们约定，需要显示字符数的名字RemainingCharacters，显示验证名字ValidationText

于是我们使用布局，直接放TextBlock，于是我们的控件做好。

有一些比较难说我没有说，请去看代码 http://git.oschina.net/lindexi/WinUX-UWP-Toolkit

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。