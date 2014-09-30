---
layout: post
title: "lcc阅读随手记录"
description: ""
category:
tags: [Programming, Compiler]
---
{% include JB/setup %}

之前看EOPL感觉收获挺大，最近又花业余时间看了看编译相关的东西，这是我看lcc的时候顺手记下的一些自己的理解。这本书[《A Retargetable C Compiler》](http://book.douban.com/subject/1610344/)还挺大头的。lcc代码量不是特别大，更复杂的是tinyCC，tinyCC甚至可以直接运行C代码。

## alloc.c
为了尽量的少调用系统调用，在alloc基础上封装了一下。

## sym.c
用来存储symbol，注意scope的表示方法。

## input.c

为了减少读取文件的开销，用一个buffer来缓存源文件内容。cp表示当前读取出来的字符位置，limit表示缓存的结尾字符位置，如果fillbuf一次以后仍然`cp == limit`则表示读取文件到EOF了。

注意这里的fread读取的时候是通过stdin的，但是在main.c/main_init函数的时候通过freopen将源文件重定向到了stdin。

fillbuf其实读取的时候是永远先把内容读取到buffer[MAXLINE+1]的位置，如果发现`cp < limit`就把前面剩下的内容往前移动，这样永远保证buffer足够下一次预读取,这里有点巧妙。

比较复杂的部分是处理resynch，input处理的内容是经过C语言预处理器的，这部分没有包含在这个编译器内。

## lex.c

一个完全是手写的C语言Parser，虽然只是兼容C99，但手写还是比较复杂的。`码农约架比写Parser是个体现实力的比赛。`

getchr逐个字符读取，cp就是input.c里面的当前字符。跳过BLANK，如果碰到NEWLINE则调用input.c读取下一行。

token.h看起来有很多列，这个文件被多个地方用到。是用宏来生成一些Enum里面的代码。比如token type和expr type。

gettok顾名思义在lex运行的时候不断提供一个一个的token，这主要是通过cp匹配map来判断，条件分支很多(依据当前的第一个字符)。`register unsigned char* rpc`存储当前字符。`register作为一个对编译器的提示，尽量用register来存储变量。事实上现在的编译器很多都能做auto register allocation，有的时候编译器的选择可能比人的选择更好。register在老的C代码里面可能更为常见。`

这个函数里面很多地方都用到了goto，主要是在匹配关键字的时候区分identifier。主要几大类是: number, keyword, identifier, string。 icon处理数字的前缀，fcon处理浮点数。

Lexical analyzer基本理论是自动状态机，没一个token可以根据相应的正则表达式来表示。有一些工具可以用来自动生成这些繁琐的代码，比如LEX，更新一些的有Flex和re2c。

## error.c
终于来到Parser部分了，lcc使用的是recursive-descent，很多商业的编译器都是用的这种直观的算法，事实上对于大部分语言都足够了。recursive-descent是自上而下的递归的，依据当前的token匹配语法结构。一个重要的问题是如何在处理的过程中给出适当的错误信息。error.c里面的函数test和expect用来测试下一个token是否是预期的,expect可以打印出错误信息。

## tree.c
最重要的数据结构struct tree，AST中的基本节点，包含子节点，和operator类型(比如AND，OR，NOT等）。在构建AST的时候root函数经常被用到。

## expr.c enode.c
parser的一部分，用来识别表达式。代码好复杂，和paresr有些类似，整个过程是构建AST。编译器的前端最重要的事情就是这了，后面的操作都是在这个基础上做的。为什么Scheme/Lisp的front部分比较简单，因为这货代码就和AST有些类似了，括号把一个一个的节点组合了起来。初看起来很难看，其实习惯了还好。

上面说的是语法的识别，在构建AST的过程中另外一个事情就是语意的分析。包括类型检查，类型的转换，操作符优先级等，这些也在构建AST的时候顺便做了。比如在遇到expr1 ? expr2 : expr3的时候，expr1的值最后被cast成一个bool。指针之间的隐式转换也比较复杂。function call比较复杂，这里还做了函数参数的写法是否是老的风格，类型说明放在函数头的最后。assignments和binary operator的分析相对来说简单一些，需要做各种cast。

前些天稍微看了一些Erlang，发现里面的类型推导比较好玩，甚至可以发现一些代码里面的逻辑错误：

比如：

{% highlight erlang %}
fact(0) -> 1;
fact(N) -> N * fact(N-1).

test() -> fact(-5).
{% endhighlight %}

不用运行Erlang的dialyzer就可以发现这里面的死循环，因为可以通过上面的定义推断出fact的参数是non_neg_integer,而-5是不符合的，所以报出来一个错误：

`fact(-5) will never return`。

## stmt.c
codelist为双向列表，遇到新的执行块就加到这个列表上。在处理control-flow的过程中有的死代码块是可以被编译器发现的，只是我们平时都被忽略了。

比如C代码:
{% highlight erlang %}
int loop() {
 Loop:
    goto Loop;
    return -1;
}

int main() {
    printf("loop: %d\n", loop());
    return 0;
}
{% endhighlight %}
loop永远不会返回，Gcc选项`-Wsuggest-attribute=noreturn`可以报出一个warning。

## decl.c
声明是C语言中最难解析的部分，原因是声明涉及到变量和类型，而从C声明中弄出类型信息还是挺复杂的。另外声明还分局部，全局，其中还涉及到函数参数，结构体等。decl.c可能是最复杂的文件了，1100多行代码，里面的函数之间又相互调用。finalize()函数最后检查是否有重复定义的变量。

## dag.c
lcc的intermediate code是用listnodes把前面parser的tree转换为DAG，最终整个程序会经过转换变成由多个DAG组合成的森林。listnodes还负责把一些公共的sub-expression简化。

接口为gencode,emitcode。后面每一个代码生成的后端都是一个Interface结构，在function函数里面调用这两个函数生成汇编代码，其中还包含一个Xinterface成员，这是平台相关的接口。


## 小结
到现在我只是大概看了了前端和中间层，后面lcc跨平台的指令生成还没来得及研究，这本书的电子版不是很清晰，还是买个中文版来再稍微看看。总的来说，lcc是的Parsing和语义分析是同时进行的，就是所谓的one-pass方法。现在很多编译器所用的方法是先建立AST，后面可能要多次遍历整个AST进行分析，LLVM好像就是采用的这种方案。另外代码的优化是一个trade-off，作为教学用途的lcc没有过多做代码优化，这样lcc代码还是可以花不多的时间来一个大概的学习。
