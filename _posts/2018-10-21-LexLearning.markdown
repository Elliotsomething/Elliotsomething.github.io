---
layout:     post
title:      "词法分析器"
subtitle:   "Lex Learning"
date:       2018-10-21
author:     "Elliot"
header-img: "img/post-bg-ios9-web.jpg"
catalog:    true
tags:
    - 词法分析
---

**版权声明：本文皆摘抄自网络，仅用于学习参考；如有侵权，请随时联系。**

编译过程的第二步，词法分析，从文件中读出数据，然后根据特定的规则，分割字符串，以便于后面的语法分析

词法分析器先对每个单词进行分类使用enum Token定义了5个种类：代码结束的eof；def；extern;identifier和number。

主要的作用就是循环读取每个词 ，然后进行分类，当然数字部分比较复杂，因为可能是0.1.1这种，需要建立报错机制。但是例子中没有给出。分类是使用c++中isdigit，isalpha之类的函数。对了，特殊符号是直接返回ascii码，比如“+”返回ascii码，定义ID，以后根据ID来确定是什么运算。

词法分析器对单词进行分析，然后就要将单词串起来，而这个串就是AST，这个工具就是语法分析器。

举个简单例子：`1+3*(4-1)+2`

<img src="https://Elliotsomething.GitHub.io/images/LexLearning-01.png">


这里使用Flex写一个简单的词法分析器：（Flex词法分析器生成工具）

`如果自己写一个通过状态机实现词法分析也比较简单，这个后续再讲`

首先我们需要下载安装Flex工具，然后使用Flex工具生成词法分析器
`brew install flex`

然后Flex会把.l文件变成对应的词法分析器的代码（在网上下一个lexico-c-flex，里面有一个比较完善的lexico.l文件）

```c
cd lexico-c-flex-master
flex -f lexico.l
```

然后我们会发现里面多了一个lex.yy.c文件，这个就是我们要的词法分析器源码，接着编译它，得到可执行文件a.out，这个就是我们要的词法分析器了
`gcc lex.yy.c`

测试一下，在桌面新建一个demo.c，里面写一行代码"int a= 100;”，然后链接运行，就可以得到结果了
`./a.out < ../demo.c`

接下来我们自己写一个词法分析器，之前我们已经得到了源码，那么我们根据其源码写一个简单的词法分析器，源码里比较重要的就是一个数组yy_nxt了，这个是一个状态机，输入一个字符，就可以得到一个状态，起始状态是1，输入状态和输出状态是3，一个takon语元结束状态为负数，另一个重要的数组就是yy_accept，这个可以根据状态扥到对应的ascall码，比如1对应空格符，21对应换行符。

代码如下：

```c
#include <stdio.h>
#include "yy_accept.c"
#include "yy_nxt.c"

int main(void) {
    int state = 1;
    char buf[64];
    int i=0;
    while (1) {
        char c;
        c = getchar();
        if (c == EOF) { break; }
        buf[i++] = c;
        state = yy_nxt[state][c];
        if (state<0) {
            int act = 0;
            state = - state;
            act = yy_accept[state];
            buf[i-1] = '\0';
            if (!(act == 1 || act == 21)) {
                printf("Pattern found! %s\n", buf);
            }
            i=0;
            state = 1;
            ungetc(c,stdin);
        }
    }
    return 0;
}
```
