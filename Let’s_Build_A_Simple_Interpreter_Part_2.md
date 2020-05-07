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

