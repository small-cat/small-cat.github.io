---
layout: post
title: antlr4 简介
date: 2019-10-08 20:06
tags: antlr4 语法分析

---

简单介绍 antlr4的基本知识，介绍了antlr4 语法中二义性及解决思路，anrlr4 可能出现的错误，以及错误定位和解决的办法。

# 简单介绍

ANTLR（Another Tool for Language Recognition）是一个开源的语法分析器生成工具。ANTLR4 语法分析器使用了一种名为自适应的 `LL(*)` 或者 `ALL(*)`(读作 all star)的新技术，`ALL（*）`是 ANTLR3 中 `LL(*)`的扩展。

早期 Antlr 的 `LL(*)` 文法仍不支持“左递归”（left-recursion），这是所有LL剖析器]的局限，在左递归过程没有消耗掉任何token,  LL 分析器很容易造成stack overflow。ANTLR4 的 `ALL(*)` 解决了左递归的问题，但是仍然不能处理**间接左递归的情况**[^1]

antlr4 是用 java 编写的，所以首先保证环境中 java 环境已经正确安装。在官网或者 github 下载 `antlr-4.7.1-complete.jar`，然后配置环境变量如下

```shell
# ANTLR
ANTLRPATH=/home/jona/software/antlr4/antlr-4.7.1-complete.jar
export CLASSPATH=.:$ANTLRPATH:$CLASSPATH
alias antlr4="java -Xmx1000M -cp "/home/jona/software/antlr4/antlr-4.7.1-complete.jar:$CLASSPATH" org.antlr.v4.Tool"
alias grun="java org.antlr.v4.gui.TestRig"
```

这样就能使用antlr4 工具了。antlr4 的 IDE 名为 `antlrworks2`。使用图形工具编写语法规则会更加高效。

antlr4 虽然是用 java 语言写的，但是生成的目标语言可以支持 cpp, c sharp, go, java, php, python 和 swift。在源码目录 `antl4/runtime` 中可以查看得到。antlr4 支持上写文无关文法规则(context-free)，能够根据语法规则生成相应的语法解析代码，开发者根据生成的代码，编写自己的逻辑。

antlr4 工具提供如下选项

```
 -o ___              specify output directory where all output is generated
 -lib ___            specify location of grammars, tokens files
 -atn                generate rule augmented transition network diagrams
 -encoding ___       specify grammar file encoding; e.g., euc-jp
 -message-format ___ specify output style for messages in antlr, gnu, vs2005
 -long-messages      show exception details when available for errors and warnings
 -listener           generate parse tree listener (default)
 -no-listener        don't generate parse tree listener
 -visitor            generate parse tree visitor
 -no-visitor         don't generate parse tree visitor (default)
 -package ___        specify a package/namespace for the generated code
 -depend             generate file dependencies
 -D<option>=value    set/override a grammar-level option
 -Werror             treat warnings as errors
 -XdbgST             launch StringTemplate visualizer on generated code
 -XdbgSTWait         wait for STViz to close before continuing
 -Xforce-atn         use the ATN simulator for all predictions
 -Xlog               dump lots of logging info to antlr-timestamp.log
 -Xexact-output-dir  all output goes into -o dir regardless of paths/package
```

antlr4 提供了两种访问模式，一个是访问者 visitor 模式，一个是监听器 listener 模式，`-visitor` 和 `-no-visitor` 分别是打开访问者和关闭访问者的选项，`-listener` 和 `-no-listener` 分别是打开监听器和关闭监听器的模式。`-long-messages`会显示详细的错误信息和告警信息。 `-package` 选项，会在代码生成时，制定代码所在的 namespace。其他选项可以参考官方文档。比如 

```
java -Xmx500M -cp /home/jona/software/antlr4/antlr-4.7.1-complete.jar org.antlr.v4.Tool -Dlanguage=Cpp -long-messages -listener -visitor -o generated/ KingbaseSqlLexer.g4 KingbaseSqlParser.g4
```

这里，根据词法文件 `KingbaseSqlLexer.g4` 和语法文件 `KingbaseSqlParser.g4` 生成 cpp 的语法分析器，源文件存储在 generated 目录中，同时打开了访问者和监听器模式。

关于 visitor 和 listener 的具体使用方法，可以参考[antlr4 权威指南]，这本书讲解的非常详细。下面题主想要写的，是在实际工作中所遇到的一些问题，想跟大家分享一下。

# 左递归和间接左递归

antlr4 是可以处理左递归的，但是不能处理间接左递归，这个在 [issue#417](https://github.com/antlr/grammars-v4/issues/417) 中有过讨论。

```
expr
    : expr '*' expr
    | expr '+' expr
    | id
    ;
```

上面这种情况就是左递归，expr 本身又是表达式，同时还可以是 id 标识符。但是下面这种情况就属于间接左递归了，这种情况 antlr4 还不能处理，会出现错误 `The following sets of rules are mutually left-recursive`

```
expr
    : expr1 '*' expr1
    | expr1 '+' expr1
    | id
    ;

expr1
    : expr '==' expr  // indirect left-recursion to expr rule.
    | id
    ;
```

expr 是 expr1 组成的表达式，同时，expr1 又是 expr 组成的表达式，二者相互引用，构成了相互左递归。这种情况必须通过优化语法结果的方式消除，antlr4 才能正确的生成语法分析的代码。

举一个明显一点的例子，下面这种情况的间接左递归

```
table_ref
	: limit_clause
	| join_clause
	;
	
limit_clause
	: table_ref limit_clause_part
	;

join_clause
	: table_ref join_clause_part
	;
```

通过优化语法，`limit_clause` 和 `join_clause` 有很多共同的部分，把相同的部分提取出来，不同的部分作为两个分支处理，可以改为下面这种方式

```
table_ref
	: table_ref (limit_clause_part | join_clause_part)
	;
```

这样就正确的消除了左递归。antlr4 是可以处理右递归的。

# 二义性和两种消除二义性的方法

## token 引起的二义性(Lexer)

比如关键字 `async`是一个token，有如下这样一条语句

```
async var async = 42;
```

在这句话中，`async`既是一个关键字，同时还是一个变量，这就出现了二义性的问题。这种情况 antlr4 有两种方法解决：

1. 在语法规则中增加语义判定

   ```
   async: {_input.LT(1).GetText() == "async"}? ID ; 
   ```

   如果 `async` 关键字存在，那么就是一个关键字，如果不存在，就是ID， 就是一个标识符。但是这种方法，使得代码与规则发生了耦合，不利于规则的维护。antlr4 相比于前面的版本，就是实现了代码与规则的解耦，使得代码与语法规则能够相互独立分开，易于维护和阅读。

2. 直接将该 token 插入到 id 的定义中

   ```
   ASYNC: 'async';
   ...
   id
   : ID
   ...
   | ASYNC;
   ```

   这样，标识符中包含了 `async`，就能正确表示了。
   
   ## 表达式中的二义性(Parser)
   
   比如下面这个语法规则
   
   ```
   stat: expr ';' // expression statement
       | ID '(' ')' ';' // function call statement;
       ;
   expr: ID '(' ')'
       | INT
       ;
   ```
   
   当 `ID '(' ')'` 出现时，我们不能确定，这是一个 `expression statement` 还是一个 `function call statement`，这就造成了二义性。
   
   ANTLR4 在生成此法分析器的过程中是不能检测二义性的，但是如果我们设定模式ALL(ALL 是一种动态算法 dynamic algorithm)，在分析过程中是可以确定二义性的。二义性可能出现在词法分析中，也可能出现在语法分析中，词法分析中的二义性的情况就是上一小节的情况，语法分析就是当前小节的情况。然而，对于一些语言(比如 c++)中，可以允许接受的一些二义性的情况，可以通过增加语义判定的方式解决(semantic predicates code insertions to resolve)，比如下面这种方式
   
   ```
   expr: { isfunc(ID) }? ID '(' expr ')' // func call with 1 arg
       | { istype(ID) }? ID '(' expr ')' // ctor-style type cast of expr
       | INT
       | void
       ;
   ```
   
   通过判定 `ID` 是 func 还是 expr，来决定是函数调用还是表达式。
   
   > 在 c++ 语法中，之前的版本有一个问题，就是 >> 的问题，>> 是一个右移运算符，同时，对于 `std::vector<std::list<std::string>>` 这种情况，最后面也出现了 >> 的符号，这个时候就出现了二义性的问题，这个方法是怎么解决的呢，**查看资料**

# 几种常见的规则调试手段

## ANTLR4 中的几种错误

- **Token recognition error** (Lexer no viable alt). Is the only lexical error, indicating the absence of the rule used to create the token from an existing lexeme:

  class **#** { int i; } — **#** is the above mentioned lexeme.

- **Missing token.** In this case, ANTLR inserts the missing token to a stream of tokens, marks it as missing, and continues parsing as if this token exists.

  class T { int f(x) { a = 3 4 5; } **}** — **}** is the above mentioned token.

- **Extraneous token.** ANTLR marks a token as incorrect and continues parsing as if this token doesn’t exist: The example of such a token will be the first **;**

  class T **;** { int i; }

- **Mismatched input.** In this case "panic mode" will be initiated, a set of input tokens will be ignored, and the parser will wait for a token from the synchronizing set. The 4th and 5th tokens of the following example are ignored and **;** is the synchronizing token

  class T { int f(x) { a = 3 **4 5**; } }

- **No viable alternative input.** This error describes all other possible parsing errors.

  class T { **int ;** }

  当然，是可以手动在规则分支中添加错误处理的方式处理错误，如下所示

  ```
  function_call
      : ID '(' expr ')'
      | ID '(' expr ')' ')' {notifyErrorListeners("Too many parentheses");}
      | ID '(' expr {notifyErrorListeners("Missing closing ')'");}
      ;
  ```

## 在 ANTLR4 中添加自定义的错误监听器

ANTLR4 提供几种默认的错误机制，`ANTLRErrorListener` 和 `ANTLRErrorStrategy`，我们可以通过继承的方式，实现自己的错误监听器

```c++
class ErrorVerboseListener : public antlr4::BaseErrorListener {
	public:
		ErrorVerboseListener(){}
		~ErrorVerboseListener() {}
		
		void syntaxError(antlr4::Recognizer *recognizer, antlr4::Token *offendingSymbol, size_t line, size_t charPositionInLine, const std::string &msg, std::exception_ptr e);
}
```

继承和实现 syntaxError 函数，这个函数就是错误处理函数。其中，line 是错误所在行数，charPositionInLine 是所在列，msg 是详细的错误信息，offendingSymbol 是错误出现的 Token 。这些信息，能够对定位规则中出现的错误提供一定的帮助。

通过下面的方法，在 cpp 中使用错误监听器

```c++
// get a parser
ANTLRInputStream input(str);
XXXLexer lexer(&input);
CommonTokenStream tokens(&lexer);
XXXParser parser(&tokens);

// remove and add new error listeners
ErrorVerboseListener err_listener;
parser.removeErrorListeners();	// remove all error listeners
parser.addErrorListener(&err_listener);	// add
```

## 规则定位(调试)

当出现上述的 ANTLR4 错误时，可以通过以下几种方法定位问题。

**一** 

根据错误信息，也可以自定义的错误监听器提供的信息，定位错误发生的 token 或者地点，然后打印整颗语法分析树结果，如果发生错误，语法分析树会在发生错误的时候，停止解析后面的内容，通过语法分析树，可以确定前面的语法解析所分析出来的语法分支是否与预期一致

```
line 1:24 extraneous input 'FROM' expecting {ABORT, ABS, ACCESS,
```

语法分析树结构如下所示，这只是我的一个例子，原语句是对 sql 语句 `select name, phone from from student` 进行语法分析

```
(sql_script (unit_sql_statement (unit_statement (sql_statement (data_manipulation_language_statements (select_statement (subquery (subquery_basic_elements (query_block SELECT (selected_list (selected_list_element (column_name (identifier (id_expression (regular_id (non_reserved_keywords_pre12c NAME)))))) , (selected_list_element (column_name (identifier (id_expression (regular_id PHONE)))))) (from_clause FROM (table_ref_list (table_ref (table_ref_aux (table_ref_aux_internal FROM (dml_table_expression_clause (tableview_name (table_name (identifier (id_expression (regular_id STUDENT))))))))))) limit_clause))))))) ;) <EOF>)
```

可以看到，错误信息指出是在 1:24，即第1行24列处，token 为 from 时发生了错误，语法解析树解析到第二个from 时，语法分支就出现了错误，不是预期的结果。

**二** 

查看解析出来的词法 tokens ，查看 tokens 是否解析错误(有时候，tokens 解析就会发生问题，直接导致后面的语法解析出现异常，或者得不到预期的结果)

```
[@0,0:5='SELECT',<1487>,1:0]
[@1,6:6=' ',<2326>,channel=1,1:6]
[@2,7:10='NAME',<882>,1:7]
[@3,11:11=',',<2302>,1:11]
[@4,12:12=' ',<2326>,channel=1,1:12]
[@5,13:17='PHONE',<2325>,1:13]
[@6,18:18=' ',<2326>,channel=1,1:18]
[@7,19:22='FROM',<555>,1:19]
[@8,23:23=' ',<2326>,channel=1,1:23]
[@9,24:27='FROM',<555>,1:24]
[@10,28:28=' ',<2326>,channel=1,1:28]
```

我们直接看这两个 from (我对所有的字符进行了大小写敏感的转换，所以这里看到的都是大写)

```
[@7,19:22='FROM',<555>,1:19]
[@8,23:23=' ',<2326>,channel=1,1:23]
[@9,24:27='FROM',<555>,1:24]
```

@7 表示第七个位置(从0开始), 19:22 表明在第19-22和字符之间，内容是 FROM，token 的 id 是 555, 1:19 表示的是，位于输入字符串第一行，第19个位置处。

这里的 token id 是指 antlr4 生成语法分析器时，在后缀为 `XXXLexer.tokens` 文件中，各个tokens 赋予的值，上面这两个 from，第一个的 token id 是555, 第二个是 555, 在 `XXXLexer.tokens` 中，from 就是 555, 这里的词法解析是正确的

# 在 cpp 目标中，使用 LL 和 ALL 优化

> Moreover, ANTLR 4 allows you to use your own error handling mechanism. This option may be used to increase the performance of the parser: first, code is parsed using a fast `SLL` algorithm, which, however, may parse the ambiguous code in an improper way. If this algorithm reveals at least a single error (this may be an error in the code or ambiguity), the code is parsed using the complete, but less rapid ALL-algorithm. Of course, an actual error (e.g., the missed semicolon) will always be parsed using LL, but the number of such files is less compared to ones without any errors.

**`LR(*)与LL(*)`**

现在主流的语法分析器分两大阵营，LR(*)与LL(*)。

**LR**是自低向上（bottom-up）的语法分析方法，其中的**L**表示分析器从左（**L**eft）至右单向读取每行文本，**R**表示最右派生（**R**ightmost derivation），可以生成**LR**语法分析器的工具有YACC、Bison等，它们生成的是增强版的**LR**，叫做**LALR**。

**LL**是自顶向下（top-down）的语法分析方法，其中的第一个**L**表示分析器从左（**L**eft）至右单向读取每行文本，第二个**L**表示最左派生（**L**eftmost derivation），ANTLR生成的就是**LL**分析器。

**`ALL(*)原理`**

ANTLR从4.0开始生成的是**ALL(\*)**解析器，其中**A**是自适应（**A**daptive）的意思。**ALL(\*)**解析器是由Terence Parr、Sam Harwell与Kathleen Fisher共同研发的，对传统的**LL(\*)**解析器有很大的改进，ANTLR是目前唯一可以生成**ALL(\*)**解析器的工具。

**ALL(\*)**改进了传统**LL(\*)**的前瞻算法。其在碰到多个可选分支的时候，会为每一个分支运行一个子解析器，每一个子解析器都有自己的DFA（deterministic  finite  automata，确定性有限态机器），这些子解析器以伪并行（pseudo-parallel）的方式探索所有可能的路径，当某一个子解析器完成匹配之后，它走过的路径就会被选定，而其他的子解析器会被杀死，本次决策完成。也就是说，**ALL(\*)**解析器会在运行时反复的扫描输入，这是一个牺牲计算资源换取更强解析能力的算法。在最坏的情况下，这个算法的复杂度为O(n<sup>4</sup>)，它帮助ANTLR在解决歧义与分支决策的时候更加智能。

在cpp 中，按照下面所示选择使用 SLL 还是 ALL

```c++
  // PredictionMode: LL, SLL
  // try with simpler and faster SLL first
  parser.getInterpreter<atn::ParserATNSimulator>()->setPredictionMode(
      atn::PredictionMode::SLL);
  parser.removeErrorListeners();

  // add error listener
  ErrorVerboseListener err_verbose;
  parser.addErrorListener(&err_verbose);
  parser.setErrorHandler(std::make_shared<BailErrorStrategy>());

  // BailErrorStrategy 会抛出 ParseCancellationException 的异常
  try {
    std::cout << "Try with SLL(*)" << std::endl;
    _ParseString(parser, tokens);
  } catch (ParseCancellationException ex) {
    std::cout << "Syntax error, try with LL(*)" << std::endl;
    std::cout << ex.what() << std::endl;

    // rewind input stream
    tokens.reset();
    parser.reset();

    // back to default listener and strategy
    parser.addErrorListener(&ConsoleErrorListener::INSTANCE);
    parser.setErrorHandler(std::make_shared<DefaultErrorStrategy>());
    parser.getInterpreter<atn::ParserATNSimulator>()->setPredictionMode(
        atn::PredictionMode::LL);

    _ParseString(parser, tokens);
  }

```


# Reference
1. [ANTLR4进阶](https://liangshuang.name/2017/08/20/antlr/)
2. [theory and practice of souce code](http://blog.ptsecurity.com/2016/06/theory-and-practice-of-source-code.html)

---
**FootNotes**

[^1]: 间接左递归后面详细阐述

