```markdown
<!--
无需连字符
[nh[scalbn]]
[nh[scalbnf]]
[nh[scalbnl]]
[nh[scalbln]]
[nh[scalblnf]]
[nh[scalblnl]]
<!-- 不能做非字母字符的事情
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

<!-- 索引见also部分 -->
[is[字符串==>查看`char *`]]
[is[换行==>查看`\n`的换行符]]
[is[三元运算符==>查看`?:`的三元运算符]]
[is[加法运算符==>查看`+`的加法运算符]]
[is[减法运算符==>查看`-`的减法运算符]]
[is[乘法运算符==>查看`*`的乘法运算符]]
[is[除法运算符==>查看`/`的除法运算符]]
[is[取模运算符==>查看`%`的取模运算符]]
[is[逻辑非==>查看`!`运算符]]
[is[逻辑与==>查看`&&`运算符]]
[is[逻辑或==>查看`||`运算符]]
[is[响铃==>查看`\a`运算符]]
[is[制表符(更好)==>查看`\t`运算符]]
[is[回车==>查看`\r`运算符]]
[is[十六进制==>查看`0x`十六进制]]

# 前言

> _C语言不是一门庞大的语言，并且不适合于冗长的书籍。_
>
> --Brian W. Kernighan, Dennis M. Ritchie

各位，言简意赅地，让我们直接进入C代码吧：

``` {.c}
E((ck?main((z?(stat(M,&t)?P+=a+'{'?0:3:
execv(M,k),a=G,i=P,y=G&255,
sprintf(Q,y/'@'-3?A(*L(V(%d+%d)+%d,0)
```

从此，他们过上了幸福美满的生活。完。

这是什么？你说，关于整个C编程语言的事情还不够清楚？
```

```c
// 翻译成简体中文:

// 坦白说，老实讲，我其实都不太清楚上面的代码是干嘛的。
// 这是从2001年[fl[国际混淆C代码大赛|https://www.ioccc.org/]][i[国际混淆C代码大赛]]的一个条目中摘录的片段，这是一个很棒的比赛，参赛者会尽力编写尽可能难懂的C代码，常常会有令人惊讶的结果。

// 坏消息是，如果你是这方面的新手，所有你看到的C代码都可能看起来像是被混淆了！好消息是，这种情况不会持续太久。

// 在这个指南中，我们将努力带领你摆脱完全迷失和困惑，走向只有通过纯粹的C编程才能获得的那种开朗幸福感。太对了。

// 在过去，C语言是一门更简单的语言。这本书中包含的许多功能以及《库参考手册》中的许多功能，在K&R在1988年编写他们著名的第二版时并不存在。尽管如此，该语言在其核心处仍然保持紧凑，我希望我在这里呈现的方式是从这个简单的核心开始，逐步向外构建。

// 这就是我写这样一本滑稽巨著来描述如此简单精练的语言的借口。

## 受众

// 本指南假设您已经掌握了其他语言的一些编程知识，比如
[flw[Python|Python_(programming_language)]],
[flw[JavaScript|JavaScript]], [flw[Java|Java_(programming_language)]],
[flw[Rust|Rust_(programming_language)]],
[flw[Go|Go_(programming_language)]],
[flw[Swift|Swift_(programming_language)]] 等。
([flw[Objective-C|Objective-C]] 开发者很可能会轻松上手！)

// 我们会假设您知道什么是变量，循环的功能，函数的工作原理等等。
```

```c
[for whatever reason the best I can hope to provide is some honest entertainment for your reading pleasure] -> 无论出于何种原因，我能提供的最好可能就是一些诚实的娱乐，带给你阅读的愉悦。  
[tutorial volume] -> 教程卷！  
[library reference|https://beej.us/guide/bgclr/] ->[库参考|https://beej.us/guide/bgclr/]  
[ISO-standard C|ANSI_C] -> [ANSI C]  
[POSIX|POSIX] -> [POSIX]  
[fl[Visual Studio Community|https://visualstudio.microsoft.com/vs/community/]] -> [Visual Studio Community|https://visualstudio.microsoft.com/vs/community/]  
[fl[WSL|https://docs.microsoft.com/en-us/windows/wsl/install-win10]] -> [WSL|https://docs.microsoft.com/en-us/windows/wsl/install-win10]  
[XCode|https://developer.apple.com/xcode/] -> [XCode|https://developer.apple.com/xcode/]  
```

有许多编译器，几乎所有的编译器都可以用于这本书。C++编译器可以编译许多（但不是全部！）C代码。最好使用适当的C编译器。

## 官方主页

这份文档的官方位置是[fl[https://beej.us/guide/bgc/|https://beej.us/guide/bgc/]]。也许将来会改变，不过更可能的是所有其他指南都会从奇科州立大学的计算机上迁移出去。

## 电子邮件政策

一般情况下，我可以帮助解答电子邮件问题，所以请随时写信。但我不能保证一定会回复。我生活比较忙碌，有时候可能无法回答你的问题。那种情况下，我通常会直接删除这条消息。这并不是个人原因；我只是永远没有时间给出你需要的详细答复。

一般来说，问题越复杂，我回复的可能性就越小。如果在发送邮件前能够将问题缩小范围，并确保包括所有相关信息（比如平台、编译器、错误信息以及任何其他你认为可能帮助我排除故障的信息），那么你获得回复的可能性就会大大增加。

如果没有得到回复，就再努力尝试一下，尽量找出答案，如果仍然无法找到，那么再次写信给我，附上你找到的信息，希望这次足够让我能够提供帮助。

既然我已经教训了你如何写信和如何不要写信给我的方式，我想告诉你，我对这些年来指南所受到的一切赞誉都 _非常_ 感激。这真正鼓舞士气，让我很高兴听到它被用于有益的用途！`:-)` 谢谢！

## 镜像站

请随意在公开或私下环境下镜像这个网站。如果你选择公开镜像这个网站，并希望我在主页上添加链接，请给我发送邮件至[`beej@beej.us`](mailto:beej@beej.us)。

## 翻译者须知

如果你想将这份指南翻译成其他语言，请联系我[`beej@beej.us`](beej@beej.us)，我将在主页上添加你的翻译链接。欢迎在翻译中加入你的姓名和联系方式。

请注意下文中的版权和分发限制。

## 版权和分发

Beej's Guide to C 版权所有 © 2021 Brian "Beej Jorgensen" Hall。

除了下文明确例外规定的代码和翻译部分外，本作品采用知识共享署名-非商业性使用-禁止演绎 3.0 许可。要查看此许可证的副本，请访问[`https://creativecommons.org/licenses/by-nc-nd/3.0/`](https://creativecommons.org/licenses/by-nc-nd/3.0/)，或寄信至 Creative Commons, 171 Second Street, Suite 300, San
Francisco, California, 94105, USA。

对于许可证中的“禁止演绎”部分，有一个明确例外，即：本指南可以自由翻译成任何语言，前提是翻译准确，指南以完整形式重印。翻译的限制与原指南相同。翻译也可以包括翻译者的姓名和联系信息。

本文档中呈现的 C 源代码在此宣布为公有领域，完全免除任何许可限制。

鼓励教育工作者自由地向他们的学生推荐或提供这份指南的复印件。

更多信息，请联系[`beej@beej.us`](mailto:beej@beej.us)。 

## 献词

```c
// Translate the following content into Simplified Chinese

// The hardest things about writing these guides are:
// Learning the material in enough detail to be able to explain it
// Figuring out the best way to explain it clearly, a seemingly-endless iterative process
// Putting myself out there as a so-called _authority_, when really I'm just a regular human trying to make sense of it all, just like everyone else
// Keeping at it when so many other things draw my attention

// A lot of people have helped me through this process, and I want to acknowledge those who have made this book possible.
// Everyone on the Internet who decided to help share their knowledge in one form or another. The free sharing of instructive information is what makes the Internet the great place that it is.
// The volunteers at [fl[cppreference.com|https://en.cppreference.com/]] who provide the bridge that leads from the spec to the real world.
// The helpful and knowledgeable folks on [fl[comp.lang.c|https://groups.google.com/g/comp.lang.c]] and [fl[r/C_Programming|https://www.reddit.com/r/C_Programming/]] who got me through the tougher parts of the language.
// Everyone who submitted corrections and pull-requests on everything from misleading instructions to typos.

// Thank you! ♥
```