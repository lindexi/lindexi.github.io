# win10 uwp 读写csv 

CSV是一种通用的、相对简单的文件格式，被用户、商业和科学广泛应用。最广泛的应用是在程序之间转移表格数据，而这些程序本身是在不兼容的格式上进行操作的（往往是私有的和/或无规范的格式）。因为大量程序都支持某种CSV变体，至少是作为一种可选择的输入/输出格式。

<!--more-->
<!-- CreateTime:2018/2/13 17:23:03 -->


<div id="toc"></div>

## 使用库 Chilkat

### Nuget 安装

进入 https://www.nuget.org/packages/Chilkat.uwp/ 安装，或右击项目管理 Nuget 搜索 Chilkat 安装。

### 创建 CSV 

		
```csharp
Chilkat.Csv csv = new Chilkat.Csv();

```

创建标题，有些csv不需要标题，有些需要，如果需要标题，使用`csv.HasColumnNames = true;`

		
```csharp

csv.HasColumnNames = true;

bool success = csv.SetColumnName(0,"标题1");
success = csv.SetColumnName(1,"标题2 lindexi");
success = csv.SetColumnName(2,"标题3 标题是列标题");
success = csv.SetColumnName(3,"有些叫head的东西");

```

设置完列标题，我们需要放入数据，注意我们有多少列标题，就可以写多少列，如果我们只有3个标题，和一个没有标题的列，那么直接添加一个 SetColumnName 为`(3,"")` 。

		
```csharp
//假如数据有 两行，一开始是从0行开始，同样从0列开始

success = csv.SetCell(0,0,"2001");
success = csv.SetCell(0,1,"red");
success = csv.SetCell(0,2,"France");
success = csv.SetCell(0,3,"cheese");

success = csv.SetCell(1,0,"2005");
success = csv.SetCell(1,1,"blue");
success = csv.SetCell(1,2,"United States");
success = csv.SetCell(1,3,"hamburger");

```

写完我们把它保存在文件，在文件保存之前，保存为字符串

		
```csharp
string csvDoc;
csvDoc = csv.SaveToString();

```

写到了字符串，之后如何做我就不多说啦。

如果不知道文件读写，请看[win10 uwp 读写文件](http://blog.csdn.net/lindexi_gd/article/details/49007841)


https://www.example-code.com/csharp_winrt/csv_create.asp