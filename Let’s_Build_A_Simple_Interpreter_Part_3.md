# 从零开始写个简单的解释器（3）

原文：[Let’s Build A Simple Interpreter. Part 3.](https://ruslanspivak.com/lsbasi-part3/)

译文：


今天早上醒来后，我在想：“为什么我们觉得学习一项新技能这么难？”

我认为，这不仅仅是因为学习本身很难。一个重要原因可能是我们花了大量时间和精力通过阅读和观看获取知识，却没有足够的时间通过练习将知识转化为技能。以游泳为例。你可以花很多时间读几百本关于游泳的书，与有经验的游泳运动员和教练交流几个小时，观看所有的训练视频，但当你第一次跳进泳池时，还是会像石头一样沉下去。

归根结底的原因是：你认为自己对这门学科的了解程度并不重要——你必须将这些知识运用到实践中，才能将其转化为技能。为了帮助你练习，我在本系列的[第一部分](./Let’s_Build_A_Simple_Interpreter_Part_1.md)和[第二部分](./Let’s_Build_A_Simple_Interpreter_Part_2.md)中增加了练习。没错，你会在今天的文章和以后的文章中看到更多的练习，我向你保证：)

好了，我们开始讲今天的内容吧？

到目前为止，你已经学会了如何解释两个整数加减法的算术表达式，比如“7+3”或“12-9”。今天我要讲的是如何解析（识别）和解释包含任意数量加减运算符的算术表达式，例如“7-3+2-1”。

从图形上看，本文中的算术表达式可以用下面的语法图来表示：

![](./images/03/lsbasi_part3_syntax_diagram.png)

什么是语法图？**语法图**是一种编程语言的语法规则的图形化表示。语法图可以直观地告诉你哪些语句在你的编程语言中是允许的，哪些是不允许的。

语法图读起来很容易：只要按照箭头所示的路径走就可以了。有些路径表示选择，有些路径表示循环。

你可以把上面的语法图理解

为：一个术语后面有一个加号或减号，后面是另一个术语，而这个术语后面又有一个加号或减号，然后是另一个术语，以此类推。用文字来表述就是这个意思。你可能想知道什么是“术语”。在本文中，“术语”只是一个整数。

语法图有两个主要作用：

- 它们以图示的形式表示了一种编程语言的规范（语法）。
- 它们可以用来帮助你编写解析器——你可以按照简单的规则将图映射成代码。

你已经知道了，在标记流中识别一个短语的过程被称为**解析**。而解释器或编译器中执行这项工作的部分被称为**解析器**。解析也被称为**语法分析**，解析器也因此被称为**语法分析器**。

根据上面的语法图，下面的算术表达式都是有效的：
- 3
- 3 + 4
- 7 - 3 + 2 - 1

因为不同编程语言中的算术表达式的语法规则非常相似，我们可以用Python shell来“测试”我们的语法图。启动你的Python shell，自己去看看。

```python
>>> 3
3
>>> 3 + 4
7
>>> 7 - 3 + 2 - 1
5
```

结果和我们预期的一样。

“3+”这个表达式并不是一个有效的算术表达式，因为根据语法图，加号后面必须有一个术语（整数），否则就是语法错误。还是那句话，用Python shell试试，你自己试试就知道了。

```python
>>> 3 +
  File "<stdin>", line 1
    3 +
      ^
SyntaxError: invalid syntax
```

能够用Python shell做一些测试当然很好，但是让我们把上面的语法图映射到代码中，用我们自己的解释器来做测试，怎么样？

你从前面的文章([第1部分](./Let’s_Build_A_Simple_Interpreter_Part_1.md)和[第2部分](./Let’s_Build_A_Simple_Interpreter_Part_2.md))中知道，我们把解析器和解释器都放到了expr方法里面。再重述一下，解析器只是识别结构，确保它符合某些规范，而解释器在解析器成功识别（解析）了表达式之后，再对表达式进行求值。

下面的代码片段显示了与图中对应的解析器代码。语法图中的矩形框（*术语*）变成了解析整数的term方法，而*expr*方法只是按照语法图的流程进行操作：

```python
def term(self):
    self.eat(INTEGER)

def expr(self):
    # 从输入中获取第一个标记并把它当作当前标记
    self.current_token = self.get_next_token()

    self.term()
    while self.current_token.type in (PLUS, MINUS):
        token = self.current_token
        if token.type == PLUS:
            self.eat(PLUS)
            self.term()
        elif token.type == MINUS:
            self.eat(MINUS)
            self.term()
```

你可以看到，*expr*首先调用*term*方法。然后*expr*方法有一个*while*循环，这个循环可以执行0次或多次。在这个循环中，解析器会根据标记（是加号还是减号）做出选择。花点时间研究一下，确保上面的代码确实是遵循了算术表达式的语法图流程。

但解析器本身并不解释任何东西：如果它识别出一个表达式，它什么也不干，如果不行，它就会抛出一个语法错误。让我们修改一下*expr*方法并添加解释器代码：

```python
def term(self):
    """返回一个INTEGER标记值"""
    token = self.current_token
    self.eat(INTEGER)
    return token.value

def expr(self):
    """解析器/解释器"""
    # 从输入中获取第一个标记并把它当作当前标记
    self.current_token = self.get_next_token()

    result = self.term()
    while self.current_token.type in (PLUS, MINUS):
        token = self.current_token
        if token.type == PLUS:
            self.eat(PLUS)
            result = result + self.term()
        elif token.type == MINUS:
            self.eat(MINUS)
            result = result - self.term()

    return result
```

因为解释器需要对一个表达式求值，所以我们修改了term方法，让它返回一个整数值，并也修改了expr方法，在需要的地方执行加减法，返回解释的结果。虽然这段代码相当简单明了，但我建议花点时间研究一下。

现在就开始行动起来，看看解释器的完整代码。

这里是你的新版计算器的源代码，可以处理包含整数和任意数量的加减法运算符的有效算术表达式。
将上述代码保存到calc3.py文件中，或者直接从GitHub中下载。试用一下。看看它能不能处理我之前给你看的语法图中的算术表达式：

```python
# 标记（Token）类型
#
# 标记EOF(end-of-file)用于表示
# 没有更多的输入需要进行词法分析。
INTEGER, PLUS, MINUS, EOF = 'INTEGER', 'PLUS', 'MINUS', 'EOF'


class Token(object):
    def __init__(self, type, value):
        # 标记类型: INTEGER, PLUS, MINUS, or EOF
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
        # 输入的字符串，如："3 + 5", "12 - 5 + 3"
        self.text = text
        # self.pos为指向self.text的索引
        self.pos = 0
        # 当前标记实例
        self.current_token = None
        self.current_char = self.text[self.pos]

    ##########################################################
    # 分词器代码                                              #
    ##########################################################
    def error(self):
        raise Exception('Invalid syntax')

    def advance(self):
        """前移'pos'指针，设置'current_char'变量。"""
        self.pos += 1
        if self.pos > len(self.text) - 1:
            self.current_char = None  # 表示输入结束
        else:
            self.current_char = self.text[self.pos]

    def skip_whitespace(self):
        while self.current_char is not None and她她current_char.isspace():
            self.advance()

    def integer(self):
        """返回从输入中处理过来的（多位数）整数。"""
        result = ''
        while self.current_char is not None and她current_char.isdigit():
            result += self.current_char
            self.advance()
        return int(result)

    def get_next_token(self):
        """词法分析器（也叫扫描器或分词器）

        此方法负责将句子拆分成一个个标记。一次一个标记。
        """
        while self.current_char is not None:

            if selfcurrent_char.isspace():
                self.skip_whitespace()
                continue

            if selfcurrent_char.isdigit():
                return Token(INTEGER, self.integer())

            if selfcurrent_char == '+':
                self.advance()
                return Token(PLUS, '+')

            if selfcurrent_char == '-':
                self.advance()
                return Token(MINUS, '-')

            self.error()

        return Token(EOF, None)

    ##########################################################
    # 解析器 / 解释器代码                                      #
    ##########################################################
    def eat(self, token_type):
        # 比较当前标记的类型与传递进来的标记的类型，
        # 如果两者匹配，则"吃掉"（eat）当前标记，
        # 并获取下一个标记分配给self.current_token，
        # 否则会抛出异常
        if self.current_token.type == token_type:
            self.current_token = self.get_next_token()
        else:
            self.error()

    def term(self):
        """返回一个INTEGER标记值。"""
        token = self.current_token
        self.eat(INTEGER)
        return token.value

    def expr(self):
        """算术表达式解析器/解释器。"""
        # 从输入中获取第一个标记并把它当作当前标记
        self.current_token = self.get_next_token()

        result = self.term()
        while self.current_token.type in (PLUS, MINUS):
            token = self.current_token
            if token.type == PLUS:
                self.eat(PLUS)
                result = result + self.term()
            elif token type == MINUS:
                self.eat(MINUS)
                result = result - self.term()

        return result


def main():
    while True:
        try:
            # 在python3环境下

，使用“input”代替“raw_input”
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

下面是我在笔记本上运行的情况：

```bash
$ python calc3.py
calc> 3
3
calc> 7 - 4
3
calc> 10 + 5
15
calc> 7 - 3 + 2 - 1
5
calc> 10 + 1 + 2 - 3 + 4 + 6 - 15
5
calc> 3 +
Traceback (most recent call last):
  File "calc3.py", line 147, in <module>
    main()
  File "calc3.py", line 142, in main
    result = interpreter.expr()
  File "calc3.py", line 123, in expr
    result = result + self.term()
  File "calc3.py", line 110, in term
    self.eat(INTEGER)
  File "calc3.py", line 105, in eat
    self.error()
  File "calc3.py", line 45, in error
    raise Exception('Invalid syntax')
Exception: Invalid syntax
```

还记得我在文章开头提到的那些练习吗？:)

![](./images/03/lsbasi_part3_exercises.png)

- 画一个只包含乘法和除法的算术表达式的语法图，例如“7*4/2*3”。说真的，拿起笔或铅笔，试着画一个就可以了。
- 修改计算器的源代码，以解释只包含乘法和除法的算术表达式，例如“7 * 4 / 2 * 3”。
- 从头开始写一个解释器来处理像“7 - 3 + 2 - 1”这样的算术表达式。使用任何你喜欢的编程语言，不看例子，凭空写出来。当你这样做的时候，请考虑一下所涉及的组件：一个分词器，它接收一个输入并将其转换为标记流，一个解析器，它从分词器获取信息，并将其转化成标记流，以及一个解释器，在解析器成功解析（识别）了一个有效的算术表达式后，生成结果。把这些部分串起来。花点时间把你所学到的知识转化为一个可以使用的算术表达式解释器。

### 检查你是否完全理解了

1. 什么是语法图？
2. 什么是语法分析？
3. 什么是语法分析器？

嘿，你看！你一直读到最后。谢谢你今天来过，别忘了做练习。我下次再来的时候会有新的文章——敬请关注。

下面是我推荐的书单，它们对你学习解释器和编译器非常有帮助。

1. [Language Implementation Patterns: Create Your Own Domain-Specific and General Programming Languages (Pragmatic Programmers)](http://www.amazon.com/gp/product/193435645X/ref=as_li_tl?ie=UTF8&camp=1789&creative=9325&creativeASIN=193435645X&linkCode=as2&tag=russblo0b-20&linkId=MP4DCXDV6DJMEJBL)

2. [Writing Compilers and Interpreters: A Software Engineering Approach](http://www.amazon.com/gp/product/0470177071/ref=as_li_tl?ie=UTF8&camp=1789&creative=9325&creativeASIN=0470177071&linkCode=as2&tag=russblo0b-20&linkId=UCLGQTPIYSWYKRRM)

3. [Modern Compiler Implementation in Java](http://www.amazon.com/gp/product/052182060X/ref=as_li_tl?ie=UTF8&camp=1789&creative=9325&creativeASIN=052182060X&linkCode=as2&tag=russblo0b-20&linkId=ZSKKZMV7YWR22NMW)

4. [Modern Compiler Design](http://www.amazon.com/gp/product/1461446988/ref=as_li_tl?ie=UTF8&camp=1789&creative=9325&creativeASIN=1461446988&linkCode=as2&tag=russblo0b-20&linkId=PAXWJP5WCPZ7RKRD)

5. [Compilers: Principles, Techniques, and Tools (2nd Edition)](http://www.amazon.com/gp/product/0321486811/ref=as_li_tl?ie=UTF8&camp=1789&creative=9325&creativeASIN=0321486811&linkCode=as2&tag=russblo0b-20&linkId=GOEGDQG4HIHU56FQ)

