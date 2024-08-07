<!-- 关键字和关键函数不需要翻译 -->

<!-- 不连字符化 -->
[nh[scalbn]]
[nh[scalbnf]]
[nh[scalbnl]]
[nh[scalbln]]
[nh[scalblnf]]
[nh[scalblnl]]
<!-- 不能对非字母字符执行操作
[nh[atan2]]
[nh[atan2f]]
[nh[atan2l]]
-->
[nh[lrint]]
[nh[lrintf]]
[nh[lrintl]]
[nh[llrint]]
[nh[llrintf]]
[nh[llrintl]]
[nh[lround]]
[nh[lroundf]]
[nh[lroundl]]
[nh[llround]]
[nh[llroundf]]
[nh[llroundl]]

<!-- 索引参见 -->
[is[String==>请查看 'char *']]
[is[New line==>请查看 '\n' 换行]]
[is[Ternary operator==>请查看 '?:' 三元操作符]]
[is[Addition operator==>请查看 '+' 加法操作符]]
[is[Subtraction operator==>请查看 '-' 减法操作符]]
[is[Multiplication operator==>请查看 '*' 乘法操作符]]
[is[Division operator==>请查看 '/' 除法操作符]]
[is[Modulus operator==>请查看 '%' 模运算符]]
[is[Boolean NOT==>请查看 '!' 操作符]]
[is[Boolean AND==>请查看 '&&' 操作符]]
[is[Boolean OR==>请查看 '||' 操作符]]
[is[Bell==>请查看 '\a' 操作符]]
[is[Tab (is better)==>请查看 '\t' 操作符]]
[is[Carriage return==>请查看 '\r' 操作符]]
[is[Hexadecimal==>请查看 '0x' 十六进制]]

# 前言

> _C 并不是一个庞大的语言，也不需要一本庞大的书籍来解释。_
>
> --Brian W. Kernighan, Dennis M. Ritchie

在这里不需要废话，让我们直接进入 C 代码：

``` {.c}
E((ck?main((z?(stat(M,&t)?P+=a+'{'?0:3:
execv(M,k),a=G,i=P,y=G&255,
sprintf(Q,y/'@'-3?A(*L(V(%d+%d)+%d,0)
```

他们过上了幸福快乐的生活。结局。

怎么了？你说对 C 编程语言还有什么不清楚的地方吗？

说实话，我其实不太确定上面的代码是在做什么。
这是来自2001年国际混淆C代码大赛[[International Obfuscated C Code Contest](https://www.ioccc.org/)]入围作品中的一小段代码，这是一项很棒的比赛，参赛者尽力写出最难读懂的C代码，常常带来令人惊讶的结果。

坏消息是，如果你是一个初学者，所有你看到的C代码可能看起来都是混淆的！好消息是，这种情况不会持续太久。

在本指南中，我们将尽力将你从完全迷茫的状态引导到只有通过纯C编程才能获得的启蒙之境。太好了。

在过去，C是一门更简单的语言。本书中包含的许多功能以及“库参考”中的许多功能在K&R在1988年写下他们著名的第二版书时并不存在。尽管如此，这门语言核心仍然很小，我希望我在这里呈现的方式是从这个简单核心开始，然后向外构建。

这就是我为什么为这么简短的一门语言写了这么一本令人发笑的庞大书的理由。

## 受众

本指南假设你已经具有其他语言的一些编程知识，比如[Python](Python_(programming_language))、[JavaScript](JavaScript)、[Java](Java_(programming_language))、[Rust](Rust_(programming_language))、[Go](Go_(programming_language))、[Swift](Swift_(programming_language))等。([Objective-C](Objective-C) 开发者会觉得特别容易！)

我们假设你知道什么是变量，循环的作用是什么，函数如何工作，等等。

如果由于任何原因这并不适合你，我希望能为你提供一些诚实的娱乐，让你愉快地阅读。我唯一可以合理承诺的是这个指南不会以悬念结束...或者会吗？

## 阅读本书的方法

本指南分为两卷，这是第一卷：教程卷！

第二卷是 [fl[库参考手册|https://beej.us/guide/bgclr/]]，更多是参考而非教程。

如果你是新手，通常按照教程部分的顺序进行。在章节中进展得越多，按顺序阅读的重要性就越小。

无论你的技能水平如何，参考部分都列有标准库函数调用的完整示例，可随时帮助您恢复记忆。适合在吃碗谷类食品或其他时间阅读。

最后，浏览索引（如果您在阅读印刷版本），参考部分的条目是斜体字。

## 平台和编译器

我会尽量坚持使用普通的 [flw[ISO标准C|ANSI_C]]。嗯，大部分时间是这样的。偶尔我可能会疯狂地开始谈论 [flw[POSIX|POSIX]] 或其他内容，不过我们会看到的。

**Unix** 用户（例如Linux、BSD等）尝试在命令行中运行 `cc` 或 `gcc`，你可能已经安装了编译器。如果没有，搜索你的发行版了解如何安装 `gcc` 或 `clang`。

**Windows** 用户应该查看 [fl[Visual Studio Community|https://visualstudio.microsoft.com/vs/community/]]。或者，如果你想要更类似于Unix的体验（推荐！），安装 [fl[WSL|https://docs.microsoft.com/en-us/windows/wsl/install-win10]] 和 `gcc`。

**Mac** 用户应该安装 [fl[XCode|https://developer.apple.com/xcode/]]，特别是命令行工具。

有很多编译器可供选择，几乎所有这些编译器都可以用于本书。C++编译器可以编译许多（但不是全部！）C代码。最好如果能使用合适的C编译器。

## 官方主页

该文档的官方位置是[fl[https://beej.us/guide/bgc/|https://beej.us/guide/bgc/]]。也许以后会改变，但更有可能的是所有其他指南都会迁移到奇科州立大学的计算机上。

## 邮件政策

我通常可以帮助回答邮件问题，所以随时写邮件过来吧，但我不能保证回复。我的生活相当忙碌，有些时候我可能无法回答你的问题。那种情况下，我通常会直接删除邮件。这并不是针对个人，只是我永远无法拥有时间给予你所需要的详细答复。

一般来说，问题越复杂，我回复的可能性就越小。如果你在发送邮件之前能够缩小问题范围，并确保包含任何相关信息（如平台、编译器、你遇到的错误消息以及其他任何可能帮助我排查的信息），那么你就更有可能得到回复。

如果没有收到回复，就再试着深入研究一下，尝试找到答案，如果依然找不到，那么请再次写信给我，附上你找到的信息，希望这足够让我能够帮助你。

现在我已经唠叨了如何给我写信和不给我写信，我只是想告诉你，我十分感激这些年来指南所收到的所有赞誉。这真的鼓舞人心，听到它被用于善事让我感到高兴！`:-)` 谢谢！

## 镜像

欢迎您复制和镜像本网站，无论是公开还是私密。如果您公开镜像本网站并希望我在主页上链接到它，请给我发邮件至[`beej@beej.us`](mailto:beej@beej.us)。

## 翻译者须知

如果您想将指南翻译成另一种语言，请写信给我[`beej@beej.us`](beej@beej.us)，我将在主页上链接到您的翻译。欢迎在翻译中添加您的姓名和联系方式。

请注意下面的版权和分发限制。

## 版权与分发

Beej的C语言指南 版权所有 © 2021 Brian "Beej Jorgensen" Hall。

除了以下源代码和翻译的具体例外情况外，本作品在署名-非商业性使用-禁止演绎 3.0 许可协议下许可。要查看此许可协议的副本，请访问[`https://creativecommons.org/licenses/by-nc-nd/3.0/`](https://creativecommons.org/licenses/by-nc-nd/3.0/)，或发送信件至 Creative Commons，地址：171 Second Street, Suite 300, San Francisco, California, 94105，美国。

该许可的“禁止演绎”部分的一个具体例外是：本指南可以被自由翻译成任何语言，前提是翻译准确，并且指南以其全部内容转载。翻译的版权限制与原始指南相同。翻译可以包括译者的姓名和联系信息。

本文档中提供的C语言源代码已放入公共领域，并完全免除任何许可限制。

教育工作者被鼓励自由向他们的学生推荐或提供本指南的副本。

欲了解更多信息，请联系[`beej@beej.us`](beej@beej.us)。

撰写这些指南最困难的地方包括：

* 学习材料以足够的细节来解释
* 找到清晰解释的最佳方法，这是一个看似无休止的迭代过程
* 将自己公之于众，被称为“权威”，而实际上我只是一个普通人，试图像其他人一样理解这一切
* 当很多其他事情吸引我的注意力时继续进行

许多人帮助我度过这个过程，我想要感谢那些使这本书得以实现的人。

* 互联网上的每个人都决定以一种形式或另一种形式分享他们的知识。免费共享信息是使互联网成为伟大场所的原因。
* [cppreference.com](https://en.cppreference.com/) 的志愿者们提供了从规范到现实世界的桥梁。
* 在[comp.lang.c](https://groups.google.com/g/comp.lang.c) 和 [r/C_Programming](https://www.reddit.com/r/C_Programming/) 的乐于助人和有知识的朋友帮助我度过了编程语言中较困难的部分。
* 每个提供了从误导性指导到错别字等方面纠正和提交请求的人。

谢谢你们！♥