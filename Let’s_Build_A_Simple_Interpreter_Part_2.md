# 从零开始写个简单的解释器（2）

原文：[Let’s Build A Simple Interpreter. Part 2.](https://ruslanspivak.com/lsbasi-part2/)

译文：

在《有效思考的5大元素》一书中，作者 Burger 和 Starbird 分享了一个故事，讲述了国际知名的小号演奏家托尼·普洛格如何在大师班中指导学生的经历。学生们首先演奏了复杂的段落，他们表现得非常出色。但当他们被要求演奏非常简单的音符时，这些音符听起来很幼稚。而当托尼演奏同样的音符时，这些音符却显得不再幼稚。托尼解释说，掌握简单音符的演奏有助于在演奏复杂乐曲时更有控制力。这堂课表明，要培养高超的演奏技巧，必须首先专注于基本理念的掌握。

这个道理不仅适用于音乐，也适用于软件开发。它提醒我们，即使有时看似在倒退，也不要忽视对基本概念的深入理解。熟练掌握工具或框架固然重要，但了解其背后的原理同样关键。正如拉尔夫·沃尔多·爱默生所说：

>"如果你只学方法，就会被方法束缚。但如果你学的是原理，你可以发明出自己的方法。"

接下来，我们将再次深入研究解释器和编译器。

今天，我将展示一个新版的计算器，它将能够：

1. 处理输入字符串中的任意空格字符。
2. 处理输入中的多位数整数。
3. 实现整数的加法和减法。

以下是新版计算器的源代码：

```python
# 标记（Token）类型
#
# 标记EOF(end-of-file)用于表示
# 没有更多的输入需要进行词法分析。
INTEGER, PLUS, MINUS, EOF = 'INTEGER', 'PLUS', 'MINUS', 'EOF'


class Token(object):
    def __init__(self, type, value):
        # 标记类型: INTEGER, PLUS, or EOF
        self.type = type
        # 标记的值: 非负整数, '+', '-', 或者 None
        self.value = value

    def __str__(self):
         """类实例的字符串表示。

        例子：
            Token(INTEGER, 3)
            Token(PLUS '+')
        """
        return 'Token({type}, {value})'.format(
            type=self.type,
            value=repr(self.value)
        )

    def __repr__(self):
        return self.__str__()


class Interpreter(object):
    def __init__(self, text):
        # 输入的字符串，如： "3+5"，"12 - 5",
        self.text = text
        # self.pos为指向self.text的索引
        self.pos = 0
        # 当前标记的实例
        self.current_token = None
        self.current_char = self.text[self.pos]

    def error(self):
        raise Exception('Error parsing input')

    def advance(self):
        """前移'pos'指针，设置'current_char'变量。"""
        self.pos += 1
        if self.pos > len(self.text) - 1:
            self.current_char = None  # 表示输入结束
        else:
            self.current_char = self.text[self.pos]

    def skip_whitespace(self):
        while self.current_char is not None and self.current_char.isspace():
            self.advance()

    def integer(self):
        """返回从输入端处理过来的（多位数）整数。"""
        result = ''
        while self.current_char is not None and self.current_char.isdigit():
            result += self.current_char
            self.advance()
        return int(result)

    def get_next_token(self):
        """词法分析器（也叫做扫描器或者分词器）

        此方法负责将句子拆分成一个个标记。一次一个标记。
        """
        while self.current_char is not None:

            if self.current_char.isspace():
                self.skip_whitespace()
                continue

            if self.current_char.isdigit():
                return Token(INTEGER, self.integer())

            if self.current_char == '+':
                self.advance()
                return Token(PLUS, '+')

            if self.current_char == '-':
                self.advance()
                return Token(MINUS, '-')

            self.error()

        return Token(EOF, None)

    def eat(self, token_type):
        # 比较当前标记的类型与传递进来的标记的类型，
        # 如果两者匹配，则 "吃掉"（eat）当前标记，
        # 并获取下一个标记分配给self.current_token，
        # 否则会抛出异常。
        if self.current_token.type == token_type:
            self.current_token = self.get_next_token()
        else:
            self.error()

    def expr(self):
        """解析器/解释器

        expr -> INTEGER PLUS INTEGER
        expr -> INTEGER MINUS INTEGER
        """
        # 从输入中获取第一个标记并把它当作当前标记
        self.current_token = self.get_next_token()

        # 我们期望当前的标记是一个整数
        left = self.current_token
        self.eat(INTEGER)

        # 我们期望当前的标记是一个'+'或者'-'号
        op = self.current_token
        if op.type == PLUS:
            self.eat(PLUS)
        else:
            self.eat(MINUS)

        # 我们期望当前的标记是一个整数
        right = self.current_token
        self.eat(INTEGER)
        # 经过所有上述过程之后self.current_token会被设为EOF标记

        # 至此所有的标记都被成功地按照 INTEGER PLUS INTEGER 或者 
        # INTEGER MINUS INTEGER的顺序找到
        # 方法只需返回两个整数相加或者相减的结果。
        # 这样我们就成功实现了对终端输入的解释
        if op.type == PLUS:
            result = left.value + right.value
        else:
            result = left.value - right.value
        return result


def main():
    while True:
        try:
            # 在python3环境下，使用“input”代替“raw_input”
            text = raw_input('calc> ')
        except EOFError:
            break
        if not text:
            continue
        interpreter = Interpreter(text)
        result = interpreter.expr()
        print(result)


if __name__ == '__main__':
    main()
```

将上述代码保存为 calc2.py 文件，或者直接从[GitHub](https://github.com/rspivak/lsbasi/blob/master/part2/calc2.py)下载。运行代码，验证其功能是否符合预期：处理输入中的空白字符，接受多位数的整数，并能进行加减法运算。

以下是我在笔记本电脑上运行的示例：

```sh
$ python calc2.py
calc> 27 + 3
30
calc> 27 - 7
20
calc>
```

与[第1部分](./Let’s_Build_A_Simple_Interpreter_Part_1.md)相比，主要的代码变化有：

1. 重构了 get_next_token 方法，将指针增量逻辑提取到一个独立的方法中。
2. 增加了两个新方法：*skip_whitespace* 用于忽略空白字符，*integer* 用于处理多位数整数。
3. 修改了 *expr* 方法，使其除了识别 INTEGER -> PLUS -> INTEGER 序列外，还能识别 INTEGER -> MINUS -> INTEGER 序列，并能进行相应的加减法运算。

在[第1部分](./Let’s_Build_A_Simple_Interpreter_Part_1.md)中，你已经学到了**标记**和**词法解析器**的概念。今天我们将讨论**词位**、**解析**和**解析器**。

你已经了解了什么是**标记**。为了更好地理解标记，我们需要介绍词位。什么是词位？**词位**是组成标记的一串字符串。下面的图片展示了一些标记和词位的例子，帮助你更好地理解它们之间的关系：

![](./images/02/lsbasi_part2_lexemes.png)

还记得我们的 *expr* 方法吗？我之前提到过，它用于解释算术表达式。在解释之前，首先需要识别表达式的类型，是加法还是减法。这就是 *expr* 方法的核心功能：在从 *get_next_token* 方法获取的标识流中找到结构，然后对识别出的表达式进行解释，计算结果。

在标识流中寻找结构的过程，或者说识别算式的过程，称为**解析**。执行这项工作的部分称为**解析器**。

现在你知道了 *expr* 方法在解释器中负责**解析**和**解释**：首先识别标识流中的INTEGER -> PLUS -> INTEGER或INTEGER -> MINUS -> INTEGER短语，然后对其进行解释，返回结果。

现在是练习时间：

![](./images/02/lsbasi_part2_exercises.png)

1. 将计算器扩展到能处理两个整数的乘法运算。
2. 将计算器扩展到能处理两个整数的除法运算。
3. 修改代码以处理包含任意数量加减法的表达式，例如 "9 - 5 + 3 + 11"。

### 检查你的理解

1. 什么是词位？
2. 在标记流中找到结构的过程叫什么？
3. 解释器或编译器中进行解析的部分叫什么？

希望你喜欢今天的课程。在本系列的下一篇文章中，你将扩展计算器以处理更复杂的算术表达式，敬请期待。

推荐书单：

1. [Language Implementation Patterns: Create Your Own Domain-Specific and General Programming Languages (Pragmatic Programmers)](http://www.amazon.com/gp/product/193435645X/ref=as_li_tl?ie=UTF8&camp=1789&creative=9325&creativeASIN=193435645X&linkCode=as2&tag=russblo0b-20&linkId=MP4DCXDV6DJMEJBL)
2. [Writing Compilers and Interpreters: A Software Engineering Approach](http://www.amazon.com/gp/product/0470177071/ref=as_li_tl?ie=UTF8&camp=1789&creative=9325&creativeASIN=0470177071&linkCode=as2&tag=russblo0b-20&linkId=UCLGQTPIYSWYKRRM)
3. [Modern Compiler Implementation in Java](http://www.amazon.com/gp/product/052182060X/ref=as_li_tl?ie=UTF8&camp=1789&creative=9325&creativeASIN=052182060X&linkCode=as2&tag=russblo0b-20&linkId=ZSKKZMV7YWR22NMW)
4. [Modern Compiler Design](http://www.amazon.com/gp/product/1461446988/ref=as_li_tl?ie=UTF8&camp=1789&creative=9325&creativeASIN=1461446988&linkCode=as2&tag=russblo0b-20&linkId=PAXWJP5WCPZ7RKRD)
5. [Compilers: Principles, Techniques, and Tools (2nd Edition)](http://www.amazon.com/gp/product/0321486811/ref=as_li_tl?ie=UTF8&camp=1789&creative=9325&creativeASIN=0321486811&linkCode=as2&tag=russblo0b-20&linkId=GOEGDQG4HIHU56FQ)
