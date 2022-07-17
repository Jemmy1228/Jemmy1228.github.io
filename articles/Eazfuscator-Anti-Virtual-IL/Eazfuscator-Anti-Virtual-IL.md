# Eazfuscator.NET虚拟机壳还原方法
## 前言
### 废话
我开始分析Eazfuscator的原因真的是好奇葩……
Eazfuscator是一个.Net程序的保护壳，而且据 @Kido 和 @wwh1004 大神说，这个壳还算是比较强的一个……
说来真的很惭愧，我很久之前就在电脑里装了Eazfuscator.NET 2018.1破解版，但是我并不知道Eazfuscator是一个强壳，一直都只用了它默认的模式（好像是有字符串加密和名称混淆），没发现它的特别之处。
五一的时候，手贱把VisualStudio升级到了2019版本，Eazfuscator也必须要升级到2019.1。没有Eazfuscator的激活码，于是便想到了春节的时候Kido大大发布的Eazfuscator的Keygen([点击此处](https://www.52pojie.cn/thread-860680-3-1.html))，看了帖子才知道Eazfuscator还有IL级虚拟化保护！
然鹅那时候我用户组还不够，无权下载，便连夜发了一篇精华挣积分。积分够了之后，总算把Keygen下载下来了……但是，Keygen生成的注册码对2019.1版本居然还无效！！！真的是心累……我猜想可能是注册码生成时候的参数需要改变，就想着把Kido大大的Keygen修改一下吧！然而Keygen还被Eazfuscator的IL虚拟化保护着……
因此我就从激活Eazfuscator开始，走上了还原Eazfuscator保护的不归路……

废话好长…不过和正文比起来了，这就不算长了[笑哭]

### 附件下载

附件里面有我本篇文章分析的UnpackMe1，分为两个版本：
UnpackMe1.clear.exe是未经混淆的，可以用来看原始IL代码
UnpackMe1.obfs.exe是受Eazfuscator.NET虚拟化保护的，也是本文主要的分析对象

另外一个BreakPoints.xml，是动态调试用到的断点，可以直接导入dnSpy-x86，(工具栏-调试-窗口-断点 Ctrl+Alt+B)
断点调试法的断点已经默认启用，IL反推法的断点需要手动设为启用

最后还有各个方法的IL代码，以及Eazfuscator的虚拟IL代码文本文件，对照着看帮助理解！

### 分析工具
对象：上面下载的UnpackMe
软件：只要dnSpy就够了！也可以再多一个ILSpy开与dnSpy对照着看。(或者用Notepad++打开附件里的il)
其他：还有就是要对.Net平台的IL中间语言有所了解，不然还原了虚拟机保护也没有用的

### 更新说明
目前为止文章里写出的研究结果已经足够分析Eazfuscator.NET的虚拟化IL保护了。
<font color=red>如果以后有新的研究结果会在文章中更新修改。</font>

## Eazfuscator.NET保护介绍
我之前没发现IL级虚拟化保护，就是因为没看过Eazfuscator的文档。我现在已经把文档***很不仔细地***读过两遍了，接下来介绍一下Eazfuscator主要用到的几种保护措施。如果说错了别喷我ORZ……
*以下的例子都可以在UnpackMe里找到，大家可以dnSpy开混淆过的UnpackMe，ILSpy开没混淆的UnpackMe对照起来看。*

### 名称混淆
这是.Net保护最基本的操作了……代码中所有的类名、方法名、变量名混淆成没有意义的字符串，让人不能直接看出代码的作用。这个就不举例子了

### 数据(资源)加密
对于字符串、数字、数组等等数据进行加密，在运行过程中还原。例如
```csharp
// UnpackMe1.MainForm
private void BtnStringEncryption_Click(object sender, EventArgs e)
{
	MessageBox.Show("The strings in this method is encrypted!", "String Encryption", MessageBoxButtons.OK, MessageBoxIcon.None);
}
```
加密后变为
```csharp
// UnpackMe1.MainForm
private void BtnStringEncryption_Click(object sender, EventArgs e)
{
	MessageBox.Show(\u0003\u2004\u2001.\u0002(2084363179), \u0003\u2004\u2001.\u0002(2084362362), MessageBoxButtons.OK, MessageBoxIcon.None);
	if (6 == 0)
	{
	}
}
```

### IL虚拟化保护
此保护就是本文的分析重点了。这种保护又有点类似于IL代码的解释执行，本质上还是IL。放段代码让大家感受一下：
原本的代码是这样，很简单的一个调用
```csharp
// UnpackMe1.MainForm
private void BtnCallVirtualization_Click(object sender, EventArgs e)
{
	MessageBox.Show("Leave CallVirtualization");
	BtnVirtualization_Click(sender, e);
	MessageBox.Show("Back to CallVirtualization");
}
```
虚拟化之后
```csharp
// UnpackMe1.MainForm
private void BtnCallVirtualization_Click(object sender, EventArgs e)
{
	object[] array = new object[3];
	object[] array2;
	if (-1 != 0)
	{
		array2 = array;
	}
	array2[0] = this;
	array2[1] = sender;
	array2[2] = e;
	\u0002\u2007 u0002_u = \u0006\u2004\u2001.\u0002\u2005\u2001();
	Stream u = \u0006\u2004\u2001.\u0003\u2005\u2001();
	string u2 = "@]iC5AnPXc";
	object[] u3 = array2;
	if (!false)
	{
		u0002_u.\u0002(u, u2, u3);
	}
}
```
呵呵，这保护完的代码亲妈都不认识……

### 代码乱序
Kido大大在他的帖子里说Eazfuscator的激活算法被乱序了，而没有被虚拟化。Eazfuscator的文档里也说有*"Code Control Flow Obfuscation"*，但是我觉得，乱不乱序好像没有区别欸……我并没有感觉到代码乱序的存在和作用。
一定要说有乱序的话，那只能认为，动态调试的分析方法直接无视了乱序吧……

## Eazfuscator.NET还原思路
Eazfuscator的保护没有传说中那么强，但是保护力度比一般的壳是要强一些的。
我觉得不太可能完全还原为最初的IL，也不容易修改代码流程(比如直接return true这种)
不管怎么说，Eazfuscator也是一个虚拟机壳啊，不要幻想着把整个程序完全能脱出来，我们只要把关键部分的的代码流程分析明白就够了。

还原的思路我说不太清楚，很多方法都是灵光一现突然想到。
大体思路我只能提出来一两点，实在不能理解的就回帖问我吧！

### 动态调试优于静态分析
IL的和PE的虚拟机壳的效果都是相近的，对于静态分析干扰非常大，对于动态调试影响较小。

我之前尝试过静态分析(直接看dnSpy中的代码)，查看每一次函数调用并且分析作用。那叫一个恶心……代码里面用了不少接口、抽象类、委托这种东西，就是说，看到调用的函数只有一个声明，静态分析很难看到真正的函数体，根本就无从下手。
光是字符串等等的数据解密都能把人绕蒙掉。

用了动态调试之后，感受就好多了(虽然还是很恶心)……有些时候，看一下方法的输入和输出，就能大概了解到它的作用，不必再一条条看代码了。而且加密的字符串在运行时会自动解密，我们直接下断点截下来即可！

而且在我们最开始没有思路的时候，直接dnSpy单步执行，看一看局部变量的变化也能对我们分析有所启发和帮助。

### 逆向思维找到关键下断点
我们一起设想一下，假如我就是Eazfuscator虚拟机壳的作者，那么我会怎样运行虚拟化的代码呢？
我随便写一段代码
```
public bool IsActivated(string serial) {
	byte[] EncryptedData = Convert.FromBase64String(serial);
	byte[] DecryptedData = dESCryptoServiceProvider.CreateDecryptor().TransformFinalBlock(EncryptedData, 0, EncryptedData.Length);
	return Encoding.ASCII.GetString(DecryptedData) == "AAAAA";
}
```
以上这段代码里存在不少的方法调用，比如FromBase64String, TransformFinalBlock, GetString
这些存在于.Net Framework之中的方法应该是不会被虚拟化的，所以虚拟化的代码一定会调用这些方法。
那么用dnSpy的分析功能，分析这些方法被的调用的地方
我知道我的UnpackMe里用到了DESCryptoServiceProvider，那我们当做白盒测试，分析一下看看吧！
把UnpackMe1.exe拖入dnSpy-x86(非x86的那个不能调试)，在mscorlib.dll找到System.Security.Cryptography命名空间，对其中的DESCryptoServiceProvider分析，
![](https://jemmy1228.github.io/articles/Eazfuscator-Anti-Virtual-IL/images/Analyze-DESCryptoServiceProvider.png)
emmm？居然没有被使用？！是dnSpy出问题了吗？难道有办法调用函数而不被分析器察觉到吗？
当然是有的，别忘了.Net有一个很神奇的功能——Reflection
[不太了了解Reflection的童鞋点这里](https://blog.csdn.net/caosiyuan1991/article/details/19172755)
如果是用了Reflection的话，Reflection一定能被分析到，因为就算Reflection被虚拟化，还是Reflection。
那我们重新试一次，选择Reflection的MethodInfo吧
![](https://jemmy1228.github.io/articles/Eazfuscator-Anti-Virtual-IL/images/Analyze-MethodInfo.png)
分析器里面那些\u000x\u000x名称的方法就是和虚拟化调用有关的方法了。

## 虚拟IL还原 - 基本方法 (一般够用了)
接下来我介绍一下我最初分析IL虚拟机时的方法。自认为这个方法比较简单，而且有效。如果各位高手有更好的想法，希望能提出来教教我欸……

附件中有两个UnpackMe, UnpackMe1.obfs.exe是我们要分析还原的虚拟化代码，这个UnpackMe很纯净，我把名称混淆关了，用的是比较纯粹的虚拟保护。另外一个UnpackMe1.clear.exe是直接编译后没有混淆的原文件。
我建议是用dnSpy-x86打开UnpackMe1.obfs.exe显示C#代码来动态调试；旁边在用ILSpy打开UnpackMe1.clear.exe显示IL代码(或者用Notepad++打开附件中我导出的IL代码)，两边对照起来看有助于我们理解虚拟机的流程。

UnpackMe有四个按钮：
StringEncryption：解密字符串并弹窗而已，就是测试下断点用的。
VirtualizedForm：弹出一个被虚拟化的窗口，让大家分析窗口字符串的产生流程。
Virtualization：在上方文本框输入注册码，点击按钮验证注册码是否正确，验证代码被虚拟化保护。
CallVirtualization：用虚拟化代码调用Virtualization的handler，分析调用堆栈用。

### 动态调试初试 (高手直接跳过吧)
初试就是尝试一下下断点、查看局部变量，就拿最简单的字符串解密开刀了……
dnSpy里打开 UnpackMe1.MainForm.BtnStringEncryption_Click(object,EventArgs) 方法
```
private void BtnStringEncryption_Click(object sender, EventArgs e)
{
	MessageBox.Show(\u0003\u2004\u2001.\u0002(2084363179), \u0003\u2004\u2001.\u0002(2084362362), MessageBoxButtons.OK, MessageBoxIcon.None);
	if (6 == 0)
	{
	}
}
```
可以看到，Eazfuscator是调用了\u0003\u2004\u2001.\u0002(int)来获得解密的字符串
所以直接对\u0003\u2004\u2001.\u0002(int)的return下断点就可以截获字符串了。（128行）
下断点的方法和VisualStudio里一样的，点击代码行号左边就行了……
然后点上方的启动，程序就会被断下来好几次，产生一些和MainForm初始化有关的字符串。
点继续十几次之后窗口跳出来，点击StringEncryption按钮，再次断下的字符串就分别是MessageBox的Text和Caption了。

### 调用方法处下断点 (断点记录法)
接下来我不会再写那么详细了……不然文章写不完了。
代码十分分散，不方便贴图贴代码了。想要看懂的话，最好是把dnSpy打开，亲手操作一遍
我们接下来对Virtualization按钮开刀
![](https://jemmy1228.github.io/articles/Eazfuscator-Anti-Virtual-IL/images/BtnVirtualization_Click.png)
分析虚拟化代码需要抓重点，举个例子：
单步调试可以知道以上的代码中真正有效的是
```
u0002_u.\u0002(u, u2, u3);
//方法签名为\u0002(Stream, string, object[]):void
```
我们可以猜测Stream是虚拟指令流，string是指令流解密key之类的东西，object[]是方法参数(包括this,sender,eventArgs)
我们的重点是虚拟代码的执行部分，因此我们就可以忽略指令流Stream u的产生过程，忽略执行器\u0002\u2007 u0002_u的产生过程，把心思放在真正执行指令的u0002_u.\u0002(u, u2, u3);方法里

这很重要！！！我最初没有想到Reflection，就是抓重点一路摸到关键断点位置的。

在调用方法处下断点真的十分简单，因为现在知道调用会用到Reflection，我又根据经验得出，好几个有关调用的关键函数都用到了MethodBase.IsConstructor。因此我们用分析器，分析(mscorlib.dll)System.Reflection.MethodBase.IsConstructor.get()被使用的情况，就可以找到关键函数。
![](https://jemmy1228.github.io/articles/Eazfuscator-Anti-Virtual-IL/images/Analyze-MethodBase.IsConstructor.png)

断点刚开始可以多下一些，对分析器找到的4个方法全部添加方法断点（在函数入口和出口各下一个，总共8个断点），然后面再把功能重复或者不需要的断点再删掉就行了，我最后在8个断点中保留了2个
```
\u0002\u2007.\u0002(MethodBase, object, object[]) : object + 0x0000 (入口处)

\u0002\u2007.\u0002(MethodBase, bool) : \u0002\u2007 + 0x2B7 (出口处)
```

另外还有一个地方比较关键，也是我单步调试跟随找到的，就在
```
(mscorlib.dll)System.Reflection.RuntimeMethodInfo.UnsafeInvokeInternal(object, object[], object[]) : object + 0x0039 (出口处)
```

我的附件中有一个Breakpoints.xml，断点下不对的童鞋可以dnSpy导入我的断点，体验一下。

下完断点或导入断点之后，断点窗口如下：
![](https://jemmy1228.github.io/articles/Eazfuscator-Anti-Virtual-IL/images/BreakPoints-Simple.png)
只要看那三个打钩激活的断点就行了，别的断点是高级方法里用的。

断点下完之后，再次开始调试，输入框里随便输入点东西(我输了AAAAA)，点击Virtualization，程序就会被dnSpy断下来。
![](https://jemmy1228.github.io/articles/Eazfuscator-Anti-Virtual-IL/images/BP-1.png)
图中\u0002就是现在调用的方法MethodBase，然后按继续
![](https://jemmy1228.github.io/articles/Eazfuscator-Anti-Virtual-IL/images/BP-2.png)
可以看到u4就是调用的执行结果

然后不停地按继续就可以了，总结之后可以知道，每个断点处的局部变量含义：

```
//\u0002\u2007.\u0002(MethodBase, object, object[]) : object + 0x0000 (入口处)

\u0002 目标方法(要调用的函数)MethodBase
\u0003 非静态方法的this
\u0005 向目标方法传递的参数
```
```
//\u0002\u2007.\u0002(MethodBase, bool) : \u0002\u2007 + 0x2B7 (出口处)

\u0002 目标方法(要调用的函数)MethodBase
obj 非静态方法的this
array5 向目标方法传递的参数
u4 方法调用后的返回值
```
```
//System.Reflection.RuntimeMethodInfo.UnsafeInvokeInternal(object, object[], object[]) : object + 0x0039 (出口处)

this 目标方法(要调用的函数)MethodBase
obj 非静态方法的this
arguments, parameters 都是参数，暂未发现区别
```
每次断点都把收集到的信息记录下来
然后就是***连蒙带猜***地反推代码咯，比如Virtualization一路继续，记录如下(MethodBase,参数,this,返回值结合起来分析，若this不为null,我记录时就加上Instance)：
```
textBox.Instance.get_Text();
data=Base32Decode(text); //这里有点难，名字被混淆了，根据MethodBase的FullName找到目标函数，看一眼代码能分析出是Base32
new DESCryptoServiceProvider();
InitializeArray(new byte[8] { 0x1F, 0x28, 0x87, 0x08, 0xCC, 0x32, 0x22, 0x35 }, RuntimeFieldHandle);
DESCryptoServiceProvider.Instance.SetKey(new byte[8] { 0x1F, 0x28, 0x87, 0x08, 0xCC, 0x32, 0x22, 0x35 });
InitializeArray(new byte[8] { 0xE2, 0xA2, 0x83, 0xBE, 0x99, 0x76, 0x84, 0x38 }, RuntimeFieldHandle);
DESCryptoServiceProvider.Instance.SetIV(new byte[8] { 0xE2, 0xA2, 0x83, 0xBE, 0x99, 0x76, 0x84, 0x38 });
DESCryptoServiceProvider.Instance.CreateDecryptor();
CryptoAPITransform.Instance.TransformFinalBlock(data);//这里的this是上一步骤CreateDecryptor()的返回值；此步骤报错，解密出错
```
以上的只是伪代码，然后自己根据语法重写一下：
```
string serial = tbInput.Text;
byte[] bEncryptedData = Base32.FromBase32String(serial);
DESCryptoServiceProvider cryptoServiceProvider = new DESCryptoServiceProvider();
cryptoServiceProvider.Key = new byte[8] { 0x1F, 0x28, 0x87, 0x08, 0xCC, 0x32, 0x22, 0x35 };
cryptoServiceProvider.IV = new byte[8] { 0xE2, 0xA2, 0x83, 0xBE, 0x99, 0x76, 0x84, 0x38 };
byte[] bDecryptedData = cryptoServiceProvider.CreateDecryptor().TransformFinalBlock(bEncryptedData, 0, bEncryptedData.Length);
```
那就很清晰明了了啊，Virtualization的实际操作是获取输入，Base32解码以后用设定的Key,IV用DES解密。因为随便输入的数据无效，所以解密报错了。
至于将注册码验证代码转换为注册码生成代码，那就是不是本文的主题了……
提供几组注册码：前5个有效，后3个无效，但是报错位置不同
```
1 Of 5     	WYYCZJM7UUNXNLWPGWTYG89S73
2 Of 5     	8S9BZUVPPMY9J5FRYDZXXHXJL5
3 Of 5     	UVGD4CC2NMD63RA5HG6EPAAWU9
4 Of 5     	W6W5LE28GNBF5ANR3UPJTKRXP2
5 Of 5     	76WWW7Z2VXUQMVCB38CXFG9W55
N Of 5     	TUYD94XEX4UR8R26MECH78FUE8
WrongLength	TUYD94XEX4URQXEGBP46943AS3
Invalid    	AAAAAAAAAAAAAAAAAAAAAAAAAA
```
用有效的注册码就可以不报错继续分析下去。
之后的流程继续分析的话，会遇到字符串解密，调用一个System.String(Int32)的MethodBase，返回的string是重点，传入参数Int32可以不管他。（解密字符串的函数是\u0003\u2004\u2001.\u0002，在局部变量窗口可能显示不完全，因为\u0000这些都是Unicode的非可见字符）

断点记录法能够获取的信息只有 **MethodBase,参数,this,返回值** 这4项，其它的操作都需要靠你的分析和联想了。只要正确猜测、分析出哪次调用的返回值，又成为了哪次调用的参数之一，就可以还原出程序执行的虚拟代码。

不同的注册码解密字符串的结果各不相同，弹窗字符也各不相同，这是为什么呢？
因为我在之后的流程中加入了if, switch这两种判断语句。不同的判断结果会跳转到不同的分支。断点记录法仅仅能够让我们知道程序现在运行的分支的流程，而分支跳转是断点记录法无法得知的，这也是断点记录法的缺点。要想了解分支跳转的信息，就必须用高级方法。

断点记录法总结一下：

要点：
> 分析器查找使用(mscorlib.dll)System.Reflection.MethodBase.IsConstructor.get()的函数
> 在函数入口出口处下断点截取4条信息(MethodBase,参数,this,返回值)
> System.Reflection.RuntimeMethodInfo.UnsafeInvokeInternal(object, object[], object[]):object出口处下断点
> 正确地连蒙带猜还原代码。

优点：
> 只要下断点和记录，操作简单
> 在大部分没有分支跳转的情况下就够用了(我就是用断点记录法还原了Kido大大的Keygen)

缺点：
> 分析能力不够的话，可能会分析错，而且靠猜测还原代码，不太严谨
> 对于非调用语句，如判断语句(if,switch等)的分析比较困难（这一点对于分析注册算法影响还是挺大的……）、

## 虚拟IL还原 - 高级方法 (IL还原法)
Eazfuscator的IL虚拟机执行方式和.Net CLR的执行方式是很相近的。
有足够的时间和精力的话，能够从Eazfuscator的虚拟IL全部转换为.Net的正常IL。
高级方法复杂一些，要求对.NetCLR运行比较了解

### .Net CLR 执行方式
#### IL指令(OpCodes)
MSDN上面有关于OpCodes的详细介绍([点击此处](https://docs.microsoft.com/zh-cn/dotnet/api/system.reflection.emit.opcodes?view=netframework-4.8#%E5%AD%97%E6%AE%B5))(建议Opcodes, OpCodeType, OperandType这三章都看一下)
不需要对所有OpCodes烂熟于心，但是对于常用的IL都需要知道是什么意思。尤其是有关分支跳转的那些IL指令。

有一点需要注意，就是IL的参数，指令可能有不同类型的参数
有些IL没有参数，如 [add](https://docs.microsoft.com/zh-cn/dotnet/api/system.reflection.emit.opcodes.add?view=netframework-4.8), [ldc.i4.0](https://docs.microsoft.com/zh-cn/dotnet/api/system.reflection.emit.opcodes.ldc_i4_0?view=netframework-4.8), [ldloc.0](https://docs.microsoft.com/zh-cn/dotnet/api/system.reflection.emit.opcodes.ldloc_0?view=netframework-4.8), [stloc.0](https://docs.microsoft.com/zh-cn/dotnet/api/system.reflection.emit.opcodes.stloc_0?view=netframework-4.8), [br.s](https://docs.microsoft.com/zh-cn/dotnet/api/system.reflection.emit.opcodes.br_s?view=netframework-4.8)
有些IL有int8参数，如 [ldc.i4.s](https://docs.microsoft.com/zh-cn/dotnet/api/system.reflection.emit.opcodes.ldc_i4_s?view=netframework-4.8), [ldloc.s](https://docs.microsoft.com/zh-cn/dotnet/api/system.reflection.emit.opcodes.ldloc_s?view=netframework-4.8), [stloc.s](https://docs.microsoft.com/zh-cn/dotnet/api/system.reflection.emit.opcodes.stloc_s?view=netframework-4.8)
还有一些IL分别带有int16, int32, int64, float32, float64
甚至还有一些IL带有奇怪类型的参数，如 [call](https://docs.microsoft.com/zh-cn/dotnet/api/system.reflection.emit.opcodes.call?view=netframework-4.8), [box](https://docs.microsoft.com/zh-cn/dotnet/api/system.reflection.emit.opcodes.box?view=netframework-4.8)

#### .Net CLR 堆栈 (数据流向)
首先要知道.Net CLR执行IL时的几个堆栈
![](https://jemmy1228.github.io/articles/Eazfuscator-Anti-Virtual-IL/images/CLR-Overview.png)
注意这里的堆栈和PE程序对的堆栈不同，这不是对于CPU真实存在的堆栈，只是一个CLR抽象出来、CLR管理的逻辑堆栈。

实际上 FunctionParameters 和 LocalParameters 都不符合先进后出，并不是一般意义上的堆栈，我不知道该怎么称呼。把他俩当做存储数据的地方就行了。

##### Evaluation Stack (计算堆栈)
上图中最中间那个就是EvaluationStack，这个堆栈也是.Net执行时最重要的堆栈。所有指令的执行都和EvaluationStack有关。

方法A调用方法B时，.Net程序会依次把 arg0(非静态方法中为this), arg1, ..., argx 依次压入堆栈
方法B在return时，EvaluationStack中唯一的数据(只能有0个或1个)就被压入方法A的EvaluationStack

所有计算操作都是发生在EvaluationStack里的

##### Function Parameters (传入参数)
FunctionParameters好像是属于CallStack的一部分的，但是我分不清楚……就按照自己的理解瞎说了
上图中最右侧那个就是Function Parameters，这里储存了调用该方法时传递的参数。
方法A调用方法B时压入A的EvaluationStack的参数会被依次弹出，进入方法B的Function Parameters
方法B通过 ldarg.1, ldarg.s 等等指令将FunctionParameters里对应的参数压入方法B自己的EvaluationStack

##### Locals Variables (局部变量)
LocalsVariables好像也是CallStack的一部分，我也不太能分清楚……
上图中最左侧那个就是LocalsVariables，这个很好理解，就是临时存放变量的地方，在IL代码的.locals中就会定义出LocalsVariables的大小，比如UnpackMe1.clear.exe的UnpackMe1.MainForm.BtnVirtualization_Click就有如下的.locals定义
```
.maxstack 4
.locals init (
	[0] uint8[],
	[1] class [mscorlib]System.Security.Cryptography.DESCryptoServiceProvider,
	[2] uint8[],
	[3] uint8
)
```
这段IL就已经限定了此方法的LocalsVariables数量和各自的类型。

方法通过 stloc.0, stloc.s 等将EvaluationStack弹出的数据存入LocalVariables；通过 ldloc.0, ldloc.s 等将LocalVariables的数据压入EvaluationStack。
注意，ldloc不会删除LocalVariables的数据，但是stloc会弹出EvaluationStack的数据

#### .Net CLR 运行
真正的运行其实还是有点复杂的，需要JIT编译器编译成Native代码执行。但是我们可以不管那些，理解为CLR解释执行就行了。这样的话，运行过程其实挺简单的……
CLR运行的内容说到底就是一串IL指令流呗
可以想象一下，CLR运行进入一个方法时，就会获取到一串该方法的指令流Stream。CLR首先按照.locals分配LocalsVariables，准备好FunctionParameters。
执行过程就是不停地Read这个Stream获取指令，然后按照指令的要求操作三个堆栈或者跳转(跳转相当于Seek指令Stream，重新设定读取的位置)
然后就一直重复以上的执行过程，直到遇到ret指令返回计算结果。

### 将.Net CLR中的概念类比到Eazfuscator中
Eazfuscator的IL虚拟化保护，真正运行的指令还是IL指令啊。想要运行的话，就必须模仿.NetCLR的执行方式，所以从.Net CLR理解之后类比到Eazfuscator就可以了。

#### 找到Eazfuscator.NET的虚拟CLR
这个CLR肯定就是一个类嘛，不可能是一个单独的方法。其实很好找，各种线索都在指向一个类 \u0002\u2007 

在断点记录法中，下的3个断点有2个都在 \u0002\u2007 之中
在被虚拟化的 MainForm.BtnVirtualization_Click() 中也是产生了 \u0002\u2007 这样的东西。

这个奇奇怪怪的 \u0002\u2007 就是Eazfuscator的虚拟CLR了。

#### 找到产生运行指令的地方
指令流肯定要循环Read，获取到现在要执行的指令才能执行起来对吧，那我们接下来就要找到这个有循环的地方。
先就用断点记录法里设置的3个断点就够了，任意输入后点击Virtualization，断点断下后，我们看一下调用堆栈(工具栏-调试-窗口-调用堆栈)
![](https://jemmy1228.github.io/articles/Eazfuscator-Anti-Virtual-IL/images/CallStack-1.png)
然后我们就从上往下一个个看吧，直到这里
![](https://jemmy1228.github.io/articles/Eazfuscator-Anti-Virtual-IL/images/CallStack-2.png)
我们才发现有一个循环，虽然现在还看不懂具体的作用，但是这里确实就是循环执行指令，使虚拟IL连续运行的地方。
\u0002\u2007.\u0005(bool):void 这里就是一个虚拟化函数的标志，每个虚拟化的函数一定都会运行到这里。
调用堆栈里该函数的数量也体现出虚拟化调用层级的个数。
比如说，点击Virtualization，我们看到调用堆栈里的\u0002\u2007.\u0005(bool):void个数始终只有一个。
但是点击CallVirtualization，当MessageBox弹出"Leave CallVirtualization"之后，再点几次继续，就能看到调用堆栈迅速膨胀，出现两层\u0002\u2007.\u0005(bool):void，如图：
![](https://jemmy1228.github.io/articles/Eazfuscator-Anti-Virtual-IL/images/CallStack-3.png)

然后从这个地方跟着一步步单步调试，就可以发现其他类比过来的东西了。

#### Eazfuscator.NET虚拟CLR的堆栈 (数据流向)
##### 数据包装类型
Eazfuscator堆栈中的数据都被包装过，数据包装的基本类型是\u0002\u2001，然后它有几种派生类型，如图所示：
![](https://jemmy1228.github.io/articles/Eazfuscator-Anti-Virtual-IL/images/u0002u2001.png)
这些派生类型的用来包装不同的基本类型，比如：

| 数据包装类名 | 存储数据类型 |
| - | - |
| \u0002\u2003 | byte |
| \u0002\u2004 | long |
| \u0002\02005 | uint |
| \u0003\u2002 | Enum |
| \u0005\u2001 | Array |
| \u0005\u2003 | short |
| \u0005\u2005 | ulong |
| \u0005\u2006 | UintPtr |
| \u0005\u2009 | string |
| \u0006\u2001 | bool |
| \u0006\u2005 | MethodBase |
| \u0006\u2009 | object |
| \u0008\u2002 | double |
| \u0008\u2004 | sbyte |
| \u000E\u2002 | float |
| \u000E\u2003 | int |
| \u000E\u2004 | ushort |
| \u000F\u2001 | char |
| \u000F\u2005 | IntPtr |
| \u000F\u2006 | object (用于存储任意其他类型) |

##### Evaluation Stack
这个EvaluationStack被处理得好奇怪der……
EvaluationStack被分成了三部分，堆栈从顶部到底部依次为\u0003\u2003, \u0008\u2001, \u0005\u2001
当数据压入EvaluationStack的时候，堆栈中所有的数据会依次从上往下移动。弹出的时候反向移动。

##### FunctionParameters
\u000F\u2000是一个数组，数组中\u000F\u2000[x]就是ldarg.x的x
注意，在非静态方法中arg.0是this，而不是第一个参数！

##### LocalsVariables
LocalVariables全部存储在\u0005中，\u0005[x]就是ldloc.x的x


#### Eazfuscator.NET的虚拟CLR变量含义
具体含义的发现思路就不细讲了……实在是说不清楚。
主要思想就是把clear.exe的IL代码和obfs的CLR局部变量变化对照起来看就可以了。
另外就是多用分析器分析一下各个函数之间的关系。
我们在任意一个\u0002\u2007里的断点断下，然后把this展开，看到的结果如下：
![](https://jemmy1228.github.io/articles/Eazfuscator-Anti-Virtual-IL/images/u0002u2007.png)

|局部变量类型 | 局部变量名称 | 局部变量作用|
| - | - | - |
| bool | \u0002\u2000 | <未知> |
| \u0003\200A | \u0002\u2002 | <确定>指令key和参数类型的对照表(后面解释) |
| System.Reflection.Module | \u0002\u2004 | <确定>当前运行的Module(对分析无用) |
| \u000E\u2001 | \u0003 | <未知> |
| \u000E\u2001 | \u0003\u2000 | <猜测>从\u0006\u2003指令数组创建的读取器 |
| System.IO.Stream | \u0003\u2001 | <猜测>传入的指令(数据)流Stream |
| \u0002\u2001 | \u0003\u2003 | <确定>EvaluationStack的一部分 |
| System.Collections.Generic.Stack<\u0002\u2007.\u0008> | \u0003\u2004 | <未知> |
| \u0002\u2001[] | \u0005 | <确定>LocalsVariables |
| System.Collections.Generic.Stack<\u0002\u2001> | \u0005\u2001 | <确定>EvaluationStack的一部分 |
| \u0008\u2000 | \u0005\u2002 | <确定>当前正在运行的方法的一些信息 |
| uint | \u0005\u2003 | <确定>下一条指令(数据)位置（指令指针） |
| long | \u0005\u2004 | <未知> |
| System.Type[] | \u0006 | <未知> |
| object | \u0006\u2000 | <未知> |
| System.Type[] | \u0006\u2002 | <未知> |
| byte[] | \u0006\u2003 | <确定>指令(数据)流数组 |
| uint | \u0008 | <确定>指令(数据)流数组的长度 |
| System.Type | \u0008\u2000 | <未知> |
| \u0002\u2001 | \u0008\u2001 | <确定>EvaluationStack的一部分 |
| System.Collections.Generic.Stack<\u0002\u2007.\u0006\u2000> | \u0008\u2003 | <未知> |
| \u0008\u2009[] | u000E\u2000 | <未知> |
| uint? | \u000E\u2001 | <确定>分支跳转目标 |
| object[] | \u000E\u2002 | <未知> |
| uint | \u000F | <确定>当前运行指令(数据)位置 (来自\u0005\u2003) |
| \u0002\u2001[] | \u000F\u2000 | <确定>FunctionParameters |
| bool | \u000F\u2001 | <确定>返回标志(return) |

虽然虚拟CLR里很多变量的含义未知，但是这并不影响我们分析的流程……现在已了解的变量就足以还原IL代码了，其余的变量等我有空的时候再试试分析含义吧。

#### Eazfuscator.NET虚拟IL指令解析
我们上文找到了循环执行指令的地方 \u0002\u2007.\u0005(bool):void
但是这并不是指令的来源，我们仔细看一下代码
![](https://jemmy1228.github.io/articles/Eazfuscator-Anti-Virtual-IL/images/u0005.png)
上面的判断是有关return和分支跳转的，我们先不用管，可以发现每次循环都会调用this.\u000E()，那我们就跟过去看看。
![](https://jemmy1228.github.io/articles/Eazfuscator-Anti-Virtual-IL/images/u000E.png)
这里就是指令执行的重点了

看一下代码的功能，先获取\u0005\u2003指令指针，将\u0005\u2003设置到\u000F (变量含义见上表)
接下来调用了\u0003\u2000的\u0006()方法，返回值为num，设置到key。
然后\u0005\u2003指令指针增加4(设定下一条指令的位置)
最后一步是用Dictionary.TryGetValue(key)得到了一个类型为\u0002\u2007.\u0002\u2000的结构，调用结构里的\u0003方法(delegate)。

我们看到key的来源是num，num来自this.\u0003\u2000.\u0006();
其中\u0003\u2000是什么东西，在上表中我只是猜测，那我们分析一下，用分析器分析\u0003\u2000赋值

在\u0002\u2007.\u0002(object[],Type[],Type[],object[]):object中能找到这样的一串代码：
```csharp
\u0006\u2003 u0006_u = new \u0006\u2003(this.\u0006\u2003);  \\用\u0006\u2003指令(数据)流初始化类型为\u0006\u2003的变量u0006_u
try
{
	using (this.\u0003\u2000 = new \u000E\u2001(u0006_u))  \\又用刚才的u0006_u初始化类型为\u000E\u2001的变量，将变量设定为\u0003\u2000
	{
		this.\u0008 = (uint)u0006_u.\u0008\u2003\u2008\u200A\u2005\u2004\u0002();
		this.\u000F\u2001 = false;
		this.\u000E\u2001 = null;
		this.\u000F = 0u;
		this.\u0005\u2003 = 0u;
		this.\u0005();
		this.\u0006();
	}
}
```
所以看出来了吧？\u0003\u2000是一个包含了数据(指令)流的东西，所以我猜测它是从指令数组创建的读取器。

然后回到上面图片里那段代码(以下为节选)
```csharp
int num = this.\u0003\u2000.\u0006();
key = num;
\u0002\u2007.\u0002\u2000 u0002_u;
global::\u0002\u2007.\u0002.TryGetValue(key, out u0002_u);
u0002_u.\u0003(this, this.\u0002(this.\u0003\u2000, u0002_u.\u0002));
```
现在我们知道key=num=\u0003\u2000.\u0006()，先放一下，等一会再继续分析。来看最后三行
定义了一个类型为\u0002\u2007.\u0002\u2000的变量u0002_u，把Dictionary.TryGetValue的结果放进了这个变量里面
变量的类型是一个结构，如图：
![](https://jemmy1228.github.io/articles/Eazfuscator-Anti-Virtual-IL/images/u0002u2000.png)
其中有一个byte，叫做\u0002
另外有一个\u0002\u2007.\u000F，叫做\u0003
那个\u0002\u2007.\u000F是一个委托的类型，定义如下：
```
private delegate void \u000F(\u0002\u2007 \u0002, global::\u0002\u2001 \u0003);
```
所以说最后一行会把this和this.\u0002(this.\u0003\u2000,u0002_u.\u0002)作为参数，调用结构体中的delegate。
this是什么，想必大家已经有部分理解了，虽然还看不懂传递给delegate的目的……
那么我们研究一下另一个参数this.\u0002(this.\u0003\u2000,u0002_u.\u0002)
传入参数中，this.\u0003\u2000是读取器，u0002_u.\u0002是结构中的那个byte
![](https://jemmy1228.github.io/articles/Eazfuscator-Anti-Virtual-IL/images/u0002-switch.png)
这个方法简单直接地switch了结构中的byte，根据不同的case，有不同的操作。

先看case到0,7的那个代码块，指令指针增加4，返回值是用\u0002.\u0006()初始化的\u000E\u2003
来来来，考验记忆力的时候到了！
\u000E\u2003是什么？还记不记得？
\u000E\u2003是int的数据包装！
然后\u0006()熟不熟悉？有没有觉得眼熟？
结合传入参数看看，就能发现\u0002.\u0006()其实就是this.\u0003\u2000.\u0006()
再看一眼我之前说放下一会分析的key=num=\u0003\u2000.\u0006()
发现了没有？
\u0003\u2000.\u0006()就相当于从哪个指令(数据)流中读取4字节，并转换为int (并不是直接转换，4字节要调换顺序)

其他的case也是如此，列表如下：

| case | 数据类型 | 读取数据长度 | IL指令举例(没想到的打问号) |
|  - | - | - | - |
| 0,7 | int | 4 | call,ldc.i4 |
| 1 | float | 4 | ldc.r4 |
| 2,9 | ushort | 2 | ??? |
| 3 | Array | 4 * (1 + length) | ??? |
| 4,5 | byte | 1 | ??? |
| 6 | long | 8 | ??? |
| 8 | double | 8 | ldc.r8 |
| 10 | null | 0 | add,ldarg.0,stloc.0 |
| 11 | sbyte | 1 | ??? |
| 12 | ulong | 4 | 所有跳转指令(指令指针) |

有几点需要注意！

首先是case12读取的ulong，照理说ulong占8字节，而这个ulong只用4字节。
因为case12只有跳转语句会用到，Eazfuscator知道数值不会超过4字节能表示的范围，ulong读取到的结果就是新分支的指令指针\u0005\u2003

另一点是不同类型的数据读取顺序不太一样……
比如说有4个字节的数据 0xA1,0xA2,0xA3,0xA4
按照int(case0)读取，那么结果就是 0xA4,0xA1,0xA2,0xA3
按照ulong(case12)读取，那么结果是 0xA2,0xA4,0xA1,0xA3

现在就算猜，也能知道delegate是什么了吧…？
调用delegate的那句才是真正产生作用的语句，这个delegate指向一个函数，这个函数会操作虚拟机的三个堆栈。
![](https://jemmy1228.github.io/articles/Eazfuscator-Anti-Virtual-IL/images/Delegates.png)
图中是\u0002\u2007中的一些委托可能指向的函数，这些函数，每一个都相当于一个IL指令。

刚才讲的有点复杂，而且名称混淆扰乱讲解，我把重要部分的代码重写一下，再看一遍应该能懂了……
```csharp
// \u0002\u2007.\u000F (VirtualCLR.OperationDelegate)
private delegate void OperationDelegate(VirtualCLR \u0002, DataWrapper \u0003);

// \u0002\u2007.\u0002\u2000 (VirtualCLR.OpCodeStructure)
private struct OpcodeStructure
{
	public readonly byte operandType;
	public readonly OperationDelegate operate;
}

// \u0002\u2007.\u000E():void
int num = this.DirectiveReader.\u0006();
int key = num;
OpCodeStructure structure;
global::VirtualCLR.OpCodeDict.TryGetValue(key, out structure);
structure.operate(this, this.GetOperand(this.DirectiveReader, structure.operandType))
```

现在我们知道 structure.operate() 指向的函数等效替代了一个IL指令，那么现在就需要分析每一个函数等效替代的IL语句

这一步方法有很多种，我就举一个例子：
比如说，我通过静态分析算法，或者查看执行前后堆栈的变化知道下图
![](https://jemmy1228.github.io/articles/Eazfuscator-Anti-Virtual-IL/images/ldarg.0.png)
这个函数是ldarg.0，那么u0002\u2007.u0003(x)就相当于是ldarg.x咯
我们用分析器分析u0002\u2007.u0003(x)，如图
![](https://jemmy1228.github.io/articles/Eazfuscator-Anti-Virtual-IL/images/u0002u2007.u0003.png)
然后就可以顺藤摸瓜找到
```csharp
//ldarg.0
private static void \u0003\u2008(\u0002\u2007 \u0002, \u0002\u2001 \u0003)
{
	if (7 == 0)
	{
	}
	\u0002.\u0003(1);
}
```
```csharp
//ldarg.s
private static void \u0008\u2008\u2000(\u0002\u2007 \u0002, \u0002\u2001 \u0003)
{
	if (6 == 0)
	{
	}
	if (4 == 0)
	{
	}
	\u0002.\u0003((int)((\u0002\u2003)\u0003).\u0002()); //这里要根据数据包装\u0002\u2003反推，得知类型为byte，即int8，所以是ldarg.s
}
```
```csharp
//ldarg.3
private static void \u0008\u200A\u2000(\u0002\u2007 \u0002, \u0002\u2001 \u0003)
{
	if (false)
	{
	}
	\u0002.\u0003(3);
}
```
```csharp
//ldarg
private static void \u000E\u2000(\u0002\u2007 \u0002, \u0002\u2001 \u0003)
{
	if (8 == 0)
	{
	}
	if (false)
	{
	}
	\u0002.\u0003((int)((\u000E\u2004)\u0003).\u0002());//同样，根据数据包装\u000E\u2004反推得知类型是ushort，即int16，所以是ldarg
}
```
```csharp
//ldarg.2
private static void \u000F\u200A\u2000(\u0002\u2007 \u0002, \u0002\u2001 \u0003)
{
	if (5 == 0)
	{
	}
	\u0002.\u0003(2);
}
```
我数了一下，大概有207个符合 OpCodeDelegate 的函数，比MSDN上的Opcodes个数226要少……
我猜测可能是因为没有nop, break等等这些对于虚拟IL没用的指令
还有一些加载数据的指令比如ldstr被ldc.i4+call等效替代了；
跳转指令全部都是读取ulong，不再区分br和br.s等等；
当然，也有可能是我数错了……

我很好奇dnSpy能根据函数的参数、返回类型搜索函数吗？或者找到实现delegate的所有函数？
我没发现有这功能，知道的大神教教我吧……手动数数实在太low了

其实没必要分析清除每一个 OpCodeDelegate 的功能，只要分析出的结果够用就行了……
用动态调试法观察每个delegate执行前后的堆栈和指令指针变化(其实就是观察\u0002\u2007里this展开的变量)比较容易吧
静态分析就用用分析器顺藤摸瓜得了，想要静态分析明白实在太累……
把一般常用的IL指令，比如ldarg.s, ldc.i4.s, ldloc.s, stloc.s, ldarg.s, call 和分支跳转指令的 delegate找到就差不多了 (下一章会讲一下分析跳转指令的方法)

接下来的步骤，就是把指令(数据)流按照 Key,Operand 转化为对应的IL (见附件里的BtnVirtualization_Click.vil)
![](https://jemmy1228.github.io/articles/Eazfuscator-Anti-Virtual-IL/images/VIL.png)
/* */之间的是指令指针和虚拟化IL对应的十六进制数据
可以看到，每个IL指令对应的十六进制总是相同的，读取十六进制转化为key和参数的时候别忘了根据数据类型调换顺序！
至于call和callvirt的MethodBase还是需要通过断点记录法的三个断点得到。

#### 分支跳转的实现原理
我们高级的IL还原法，优势就在于可以分析分支，那就讲一下分析分支的方法：
回到 \u0002\u2007.\u0005(bool):void 就是那个while循环的地方
![](https://jemmy1228.github.io/articles/Eazfuscator-Anti-Virtual-IL/images/Au0005-2.png)
while判断的条件this.\u000F\u2001是return的标志，就是说执行虚拟return时会设置this.\u000F\u2001=true，因此退出while循环，执行break，退出那个并没有任何作用的for循环，然后按照调用堆栈逐级返回。

那个if语句就是判断跳转的地方了，当遇到 br, btrue, bfalse, ble, blt, bge, bgt 等等分支跳转语句的时候，会将 \u000E\u2001 设置为跳转的目标。
\u000E\u2001是一个比较神奇的类型，uint?，实际上是Nullable<uint>，就是可以为null的uint。
当不需要跳转的时候，保持为null，需要跳转时，将指令指针\u0005\u2003设置为此unit?的值。
所以分析跳转，只要分析哪个函数将\u000E\u2001设置为非null即可，用分析器，只找到一个函数，就是u0002(uint):void
![](https://jemmy1228.github.io/articles/Eazfuscator-Anti-Virtual-IL/images/u0002.png)
可以看出它只负责设置跳转指令，而没有判断。所以可以知道其他的跳转语句都会调用它，那么继续用分析器分析。
![](https://jemmy1228.github.io/articles/Eazfuscator-Anti-Virtual-IL/images/Analyze-u0002.png)
这些就是等效替代IL跳转指令的函数了……然后需要有耐心地一点点分析每一个函数的作用了。
实在不想分析的话，那就每次修改EvaluationStack上的数据(分别大于等于小于IL指令的参数)，调用相同的delegate，看一下是否跳转(看指令指针\u0005\u2003的变化)，总是有办法分析出跳转条件的。

IL指令反推法再总结一下

要点：
> 找到while循环的位置和通过Dictionary.TryGetValue(key)获取OpCodesStructure的地方
> 然后找到虚拟机执行器的this里面三个堆栈在哪里
> 
> 通过OpCodesStructure不同的delegate确定IL指令的种类
> 
> 或者
> 
> 通过观察每步骤IL执行前后堆栈数据和指令指针位置的变化，来反向推测这个IL的种类和功能。
> 
> 截取GetParameters的返回值，得知IL指令的参数
> 将种类和参数结合，得知此刻执行的IL

> 然后不断重复以上步骤，直到把整条方法的指令流全部还原。

优点：
> 可以得知指令流的全部信息，弥补断点记录法无法判断分支的缺点。
> 分析每步骤IL的执行，反推的代码更加准确。

缺点：
> 耗费时间
> 分析delegate等效替代的IL让人想吐，这是体力活……

不过我觉得如果写一个程序，用IL反推法分析Eazfuscator虚拟化IL，还是可以做得到的……
因为每条IL指令执行后堆栈和指针的变化总是可以看到的，
而且程序分析每个delegate等效替代的IL可能很简单！
将虚拟IL流反推成标准IL流，之后再转化为C#应该是可实现的。
但是我不会写.Net脱壳工具，可能像@wwh1004这样的牛人才能做到吧……

## 总结
文章主要就是介绍了两种分析Eazfuscator.NET虚拟化IL保护壳的方法。
基本方法断点记录法操作起来了比较简单，但是不能判定分支跳转。
高级方法IL还原法还原精确度高，但是耗费体力和耐心，对手工还原不太友好……能写出辅助工具可能会好一些。
我已经尽力讲清楚了…反复多看几遍，如果还有无法理解的地方可以回帖来问我。
其实也没什么好总结的了，只想说写了这么长的文章，好累……
