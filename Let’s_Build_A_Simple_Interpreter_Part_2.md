# 从零开始写个简单的解释器（2）

原文：[Let’s Build A Simple Interpreter. Part 2.](https://ruslanspivak.com/lsbasi-part2/)

译文：

在《有效思考的5大元素》这本精彩的书中，作者 Burger 和 Starbird分享了一个故事，讲述了他们如何观察到国际知名的小号演奏家托尼-普洛格作为有成就的小号演奏家举办大师班的故事。学生们首先演奏了复杂的段落，他们演奏得非常好。但随后，他们又被要求演奏非常基本、简单的音符。当他们演奏时，与之前弹奏的复杂段落相比，这些音符听起来很幼稚。在他们演奏完之后，主讲老师也演奏了同样的音符，但是当他演奏的时候，这些音符听起来并不幼稚。两者之间的差异让人惊叹不已。托尼解释说，掌握了简单音符的演奏，可以让人在演奏复杂的乐曲时更有控制力。这堂课很清楚地表明----要培养出真正的高超的演奏技巧，就必须首先专注于掌握简单的、基本的理念。

这个故事中的道理显然不仅适用于音乐，也适用于软件开发。这个故事很好地提醒了我们所有人，即使有时觉得在走回头路，也不要忽略了在简单的、基本的想法上深耕细作的重要性。熟练掌握你所使用的工具或框架固然重要，但了解其背后的原理也极为重要。正如拉尔夫-沃尔多-爱默生所说的那样。

>"如果你只学方法，就会被方法束缚。但如果你学的是原理，你可以发明出自己的方法。"

说到这里，让我们再次深入研究一下解释器和编译器。

今天，我将向大家展示一个新版的计算器，从第1部分开始，它将能够。

1. 处理输入字符串中的任何地方的空格字符
2. 消耗输入中的多位数整数的整数。
3. 减去两个整数(目前只能加整数)

下面是你的新版计算器的源代码，可以实现以上所有的功能：

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
        #the INTEGER MINUS INTEGER的顺序找到
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

将上述代码保存到为 calc2.py 文件，或者直接从[GitHub](https://github.com/rspivak/lsbasi/blob/master/part2/calc2.py)中下载。试着运行一下。看看它的工作结果是否与预期的一样：它可以处理输入中任何地方的空白字符；它可以接受多位数的整数，既可以减去两个整数，又可以加两个整数。

下面是我在笔记本电脑上运行的示例：

```sh
$ python calc2.py
calc> 27 + 3
30
calc> 27 - 7
20
calc>
```

与[第1部分](./Let’s_Build_A_Simple_Interpreter_Part_1.md)的版本相比，主要的代码变化是。

1. get_next_token方法被重构了一下。将增量pos指针的逻辑并入入了一个单独的方法。
2. 增加了两个方法：*skip_whitespace* 方法用于忽略空白字符，以及 *integer* 方法用于处理输入中的多位数整数。
3. *expr* 方法被修改为除了 INTEGER -> PLUS -> INTEGER 序列之外，还可以识别 INTEGER -> MINUS -> INTEGER 序列。该方法现在在成功识别了相应的短语后，还可以解释加法和减法。

在[第1部分](./Let’s_Build_A_Simple_Interpreter_Part_1.md)中，你学过两个重要的概念，即**标记**和**词法解析器**。今天我想跟大家谈一下**词位**、**解析**和**解析器**。

你已经知道了什么是**标记**。但是，为了让更好地讨论**标记**，我需要提到词位。什么是词位？**词位**是组成标记的一串字符串。在下面的图片中，你可以看到一些标记和词位的例子，希望它能让你清楚地了解它们之间的关系：

![](./images/02/lsbasi_part2_lexemes.png)

现在，还记得我们的朋友，*expr* 方法吗？我之前说过，就是在那里对算术表达式进行解释的。但是在解释一个表达式之前，你首先需要识别出它是什么样的式子，比如说是加法还是减法。这就是 *expr* 方法的根本作用：它在从 *get_next_token* 方法得到的标识流中找到结构，然后对所识别的式子进行解释，求出算术表达式的结果。

在标识流中寻找结构的过程，或者换句话说，在标识流中识别算式的过程被称为**解析**。解释器或编译器中执行这项工作的部分被称为**解析器**。

所以现在你知道了 *expr* 方法是你的解释器的一部分，在这里，**解析**和**解释**都在这里进行-- *expr* 方法首先尝试识别（**解析**）标识流中的INTEGER -> PLUS -> INTEGER或INTEGER -> MINUS -> INTEGER短语，在成功识别（**解析**）了其中一个短语后，该方法对其进行解释，并返回两个整数的加减结果给调用者。

现在又到了练习的时候了。

![](./images/02/lsbasi_part2_exercises.png)

1. 将计算器扩展到能处理两个整数的乘法运算。
2. 将计算器扩展到能处理两个整数的除法运算
3. 修改代码以解释包含任意数量的加减法的表达式，例如 "9 - 5 + 3 + 11"

### 检查你的是否完全理解了

1、什么是词位？
2. 在标记流中找到结构的过程的名称是什么，或者换个说法，在该标记流中识别某个短语的过程的名称是什么？
3. 解释器（编译器）中进行解析的部分的名称是什么？

希望大家喜欢今天的课程。在本系列的下一篇文章中，你将扩展你的计算器，以处理更复杂的算术表达式。请继续关注。

下面是我推荐的书单，它们对你学习解释器和编译器有帮助。

1. [Language Implementation Patterns: Create Your Own Domain-Specific and General Programming Languages (Pragmatic Programmers)](http://www.amazon.com/gp/product/193435645X/ref=as_li_tl?ie=UTF8&camp=1789&creative=9325&creativeASIN=193435645X&linkCode=as2&tag=russblo0b-20&linkId=MP4DCXDV6DJMEJBL)

2. [Writing Compilers and Interpreters: A Software Engineering Approach](http://www.amazon.com/gp/product/0470177071/ref=as_li_tl?ie=UTF8&camp=1789&creative=9325&creativeASIN=0470177071&linkCode=as2&tag=russblo0b-20&linkId=UCLGQTPIYSWYKRRM)

3. [Modern Compiler Implementation in Java](http://www.amazon.com/gp/product/052182060X/ref=as_li_tl?ie=UTF8&camp=1789&creative=9325&creativeASIN=052182060X&linkCode=as2&tag=russblo0b-20&linkId=ZSKKZMV7YWR22NMW)

4. [Modern Compiler Design](http://www.amazon.com/gp/product/1461446988/ref=as_li_tl?ie=UTF8&camp=1789&creative=9325&creativeASIN=1461446988&linkCode=as2&tag=russblo0b-20&linkId=PAXWJP5WCPZ7RKRD)

5. [Compilers: Principles, Techniques, and Tools (2nd Edition)](http://www.amazon.com/gp/product/0321486811/ref=as_li_tl?ie=UTF8&camp=1789&creative=9325&creativeASIN=0321486811&linkCode=as2&tag=russblo0b-20&linkId=GOEGDQG4HIHU56FQ)