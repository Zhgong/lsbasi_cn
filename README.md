# 翻译《从零开始写个简单的解释器》
原文：[Let’s Build A Simple Interpreter](https://ruslanspivak.com/lsbasi-part1/)

## [从零开始写个简单的解释器（1）](./Let’s_Build_A_Simple_Interpreter_Part_1.md)
介绍编译器，解释器，词法分析等概念，并设计一个简单的计算器，用于计算个位数加法3+5。

## [从零开始写个简单的解释器（2）](./Let’s_Build_A_Simple_Interpreter_Part_2.md)
扩展功能，忽略输入算式中的空格，可以处理多位数的整数以及增加减法功能。

## [从零开始写个简单的解释器（3）](./Let’s_Build_A_Simple_Interpreter_Part_3.md)
介绍语法图（syntax diagram）。如何语法图映射到代码中。

## [从零开始写个简单的解释器（4）](./Let’s_Build_A_Simple_Interpreter_Part_4.md)
引入“巴科斯范式 Backus-Naur Form”
- 语法中定义的每一条规则都会成为一个同名的方法，
- 可选项**(a1 | a2 | aN)** 成为 ***if-elif-else*** 语句。
- 可多次匹配的分组 **(...)*** 成为一个 ***while*** 语句，可以循环0次或更多次。