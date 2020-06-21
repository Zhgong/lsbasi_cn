# 从零开始写个简单的解释器（4）

原文：[Let’s Build A Simple Interpreter. Part 4.](https://ruslanspivak.com/lsbasi-part4/)

译文：
你是被动地学习这些文章中的材料，还是一直在积极实践？我希望你一直在积极实践。我真的是这样想的 :)

还记得孔子说过什么吗？( 其实为孔子的弟子荀子在《儒孝篇》中的名句)
>不闻不若闻之

![](./images/04/LSBAWS_confucius_hear.png)

>闻之不若见之

![](./images/04/LSBAWS_confucius_see.png)

>见之不若知之，知之不若行之

![](./images/04/LSBAWS_confucius_do.png)

在上一篇文章中，你学习了如何解析（识别）和解释含有任意数量加减运算符的算术表达式，例如 "7 - 3 + 2 - 1"。你还学习了语法图，以及如何使用它们来确定编程语言的语法。

今天你要学习的是如何解析和解释其中包含任意数量的乘法和除法运算符的算术表达式，例如 "7 * 4 / 2 * 3"。本文中的除法将是整数除法，所以如果表达式是 "9 / 4"，那么答案将是一个整数：2。

今天我还会讲不少关于另一种被广泛使用的用于指定编程语言语法的符号。它叫做**上下文无关文法**（简称**文法**）或**BNF**（巴科斯范式 Backus-Naur Form）。在本文中，我不会使用纯粹的[BNF](https://en.wikipedia.org/wiki/Backus%E2%80%93Naur_Form)符号，而更像是一种修改过的[EBNF](https://en.wikipedia.org/wiki/Extended_Backus%E2%80%93Naur_Form)符号。

这里有几个使用语法的理由：

1. 语法以简明的方式规定了编程语言的语法。与语法图不同，语法非常紧凑。在以后的文章中，你会看到我越来越多地使用语法。
2. 语法本身也是很好的文档。
3. 即使你从头开始手动编写你的解析器，语法也是一个很好的起点。很多时候，你可以按照一套简单的规则将语法转换为代码。
4. 有一套工具，叫做解析器生成器，它接受一个语法作为输入，并根据这个语法自动为你生成一个解析器。我将在后面的系列文章中讲到这些工具。

现在，我们来谈谈语法的作用机制，如何？

这里有一个描述算术表达式的语法，比如 "7 * 4 / 2 * 3"（这只是该语法可以生成的众多表达式之一）。

![](./images/04/lsbasi_part4_bnf1.png)

语法由一连串的*规则*组成，也就是所谓的*生产*。在我们的语法中，有两条规则。

![](./images/04/lsbasi_part4_bnf2.png)

一条规则由一个*非终端*组成，称为生产的**头部**或**左侧**，一个冒号，以及终端和/或非终端的序列，称为生产的**主体**或**右侧**。

![](./images/04/lsbasi_part4_bnf3.png)








下面是我推荐的书单，它们对你学习解释器和编译器非常有帮助。

1. [Language Implementation Patterns: Create Your Own Domain-Specific and General Programming Languages (Pragmatic Programmers)](http://www.amazon.com/gp/product/193435645X/ref=as_li_tl?ie=UTF8&camp=1789&creative=9325&creativeASIN=193435645X&linkCode=as2&tag=russblo0b-20&linkId=MP4DCXDV6DJMEJBL)

2. [Writing Compilers and Interpreters: A Software Engineering Approach](http://www.amazon.com/gp/product/0470177071/ref=as_li_tl?ie=UTF8&camp=1789&creative=9325&creativeASIN=0470177071&linkCode=as2&tag=russblo0b-20&linkId=UCLGQTPIYSWYKRRM)

3. [Modern Compiler Implementation in Java](http://www.amazon.com/gp/product/052182060X/ref=as_li_tl?ie=UTF8&camp=1789&creative=9325&creativeASIN=052182060X&linkCode=as2&tag=russblo0b-20&linkId=ZSKKZMV7YWR22NMW)

4. [Modern Compiler Design](http://www.amazon.com/gp/product/1461446988/ref=as_li_tl?ie=UTF8&camp=1789&creative=9325&creativeASIN=1461446988&linkCode=as2&tag=russblo0b-20&linkId=PAXWJP5WCPZ7RKRD)

5. [Compilers: Principles, Techniques, and Tools (2nd Edition)](http://www.amazon.com/gp/product/0321486811/ref=as_li_tl?ie=UTF8&camp=1789&creative=9325&creativeASIN=0321486811&linkCode=as2&tag=russblo0b-20&linkId=GOEGDQG4HIHU56FQ)