---
layout: page
title: 个人简历
---

# 吴震宇
邮箱：mblrwuzy@gmail.com 　　　联系电话：13530210085 <br>
个人网站：blog.csdn.net/honglicu123 和 blog.wuzhenyu.com.cn


## 教育背景
<table cellspacing="0">
<tr>
<td>2013.09 – 2015.07</td>
<td>北京交通大学</td>
<td>软件工程</td>
<td>硕士</td>
</tr>
<tr>
<td>2009.09 – 2013.07</td>
<td>长安大学</td>
<td>计算机科学与技术</td>
<td>本科</td>
</tr>　　　　	　　　
</table>


## 工作经历

2022.04 ~ 至今   　　　 斑马网络技术有限公司<br>
2019.07 ~ 2022.02　　　闪捷信息科技有限公司<br>
2015.08 ~ 2019.05　　　中国移动信息技术有限公司<br>
2014.06 ~ 2015.06　　　北京锐安科技有限公司

## 能力专长 

1. 掌握 c/c++，shell，熟悉汇编，熟悉 linux c/c++ 开发工具链。
2. 熟练掌握编译器前端中端。熟悉递归下降分析算法，熟练掌握上下文无关文法和 EBNF 范式，以及 antlr4 语法解析工具。
3. 熟悉编译原理和编译器架构，了解 llvm，了解 tvm。
4. 熟悉 clang-tidy 静态分析框架，熟悉 phasar 静态分析框架。
5. 熟悉 linux 系统，了解 linux 内核，了解内核裁剪、编译和调试，了解运行库的原理。

## 项目经验

**2020.06~至今　　　编译器开发工程师**<br>**主要工作：**

1. 基于 llvm clang-tidy 静态分析框架，实现 autosar cpp rules 规则的静态检查功能，完成包含 undefined behavior 在内的 20 多个重要规则检查功能的实现。
2. 完成 alios 中中间件 ros2 的工具链切换，从 gcc 切换到 clang 的交叉编译。完成 FimaEngine 工具链切换到 gcc 的交叉编译。
3. 基于 phasar 静态分析框架，实现 type state analysis 的 spin lock 检查的 demo 实现；完善 wllvm 实现对 kernel 交叉编译并输出完整的 llvm bytecode；修复 phasar 实现的 ide 框架中 type state analysis 分析的bug，过程间分析时 call-to-start 边函数传递的信息丢失导致分析结果不准确；修复 phasar 扫描 kernel llvm ir 时因为 type hierarchy graph 太大导致的 OOM；发现 llvm CflAnderson pta 算法的bug，在扫描kernel时无法退出，导致内存OOM。
4. 研究 tvm，新增 tvm byoc 对接 bmlib 的后端实现 demo；实现 tvm byoc 对接 acl 算子，并打通 byoc + ansor 对 ar 导航模型的分析和优化。
5. 基于 yocto，设计和实现针对斑马座舱和智驾系统的二级构建系统，统一 rootfs；设计和实现 yocto layer 的多工具链支持以及外部 rootfs 编译的 yocto layer 的支持，并输出一篇专利。

**2020.06~2022.02　　　基础平台引擎开发部语法组经理**<br>**描述：** 负责整个公司语法分析和语义分析的开发和维护工作。<br>**主要工作：**

1. 将语法分析和语义分析工作从动态脱敏项目和审计项目中解耦，实现业务与antlr4解析器的解耦。
2. 设计和实现语法分析统一框架，针对13种不同的关系型数据库的sql，构建统一的抽象语法树输出接口，通过访问器模式进行统一遍历。
3. 优化上下文无关文法，优化 antlr4 语法解析器。通过 ParseTreeWalker 的方式控制 DSF 的遍历深度；通过 SLL 优先的解析算法提升解析器的解析速度；通过 dfs cache 持久化的方式热启动，提升解析的速度。
4. 调研 mongodb 协议解析输出结果，设计和实现 mongodb api 调用与 mongodb shell command 之间转换的翻译器.
5. 负责整个公司统一操作系统镜像的裁剪和升级工作，使用smbash替换bourne shell，完成对登录账户访问权限和访问命令的控制。通过 kickstart 的方式自定义操作系统镜像，并新增镜像升级的方式，重装系统后完成产品和镜像的升级并保持原有的数据不变。

**2019.08~2020.05　　　动态脱敏系统**<br>**描述：** 负责sql语法分析和语义分析开发工作。<br>**主要工作：**

1. 将 oracle 语法分析由 bison 迁移到 antlr4。
2. 编写国产数据库 gaussdb，kingbase，dm 等多个数据库的 sql 语法规则，使用 antlr4 语法分析器进行分析。
3. 编写动态脱敏语义分析算法，根据自定义规则，定位 sql 中的敏感列并对 sql 进行重写。通过对查询语句构建统一的 DAG 结构，利用上下文 context，对 DAG 结构进行语义分析。
4. 重构动态脱敏语法模块，构建动态脱敏语法模块统一框架。
5. 优化脱敏select *展开算法，优化数据库脱敏算法，以预加载的方式，通过分段函数的形式将脱敏算法转换成表达式，使得改写后的脱敏语句性能提升了 50%。

**2015.08~2019.05　　　中国移动转售业务运营支撑系统**<br>**描述：** 负责全国31省转售业务运营与支撑。<br>**主要工作：**

1. 负责计费结算系统上线升级工作，完成上线任务 30 多次，系统重大升级 2 次；构建和完善转售系统自动化巡检工作。
2. 流程优化改造，使系统成功报竣率达到 99% 以上。
3. 发现并修复系统重大缺陷2个。
4. 发表期刊论文一篇。 
5. 担任 tap3 工具开发项目负责人，学习和研究 ASN.1 Compiler 开源项目，开发转售tap3话单的编解码引擎。

**2018.02~2018.12　　　虚拟运营商BOSS系统　　　　第四小组组长**<br>**描述：** 虚拟运营商业务运营支撑系统，为移动转售企业提供集中化平台服务。<br>**主要工作：** 

1. 负责解码模块的设计和开发，以ASN.1开源项目解码引擎作为基础，对转售系统的tap3话单进行解码。
2. 负责计费流程和余额模块的设计和开发，设计数据结构和接口。
3. 管理和跟进计费模块和余额模块的开发进度，并负责模块的验收工作。

**2016.02~2017.01　　　移动转售业务话单稽核系统　　　　项目负责人**　<br>**描述：** 本项目主要针对的是31个省份和内容计费系统以及ARCH上传给转售平台的话单，检测话单记录的有效性，筛选异常话单,进行减免和通知省侧重传，并统计转售业务每日/每月的活跃用户数。<br>**主要工作：** <br>

1. 参与和主导稽核系统的开发。采用 mysql 和 redis 进行数据存储和相关统计处理，日志使用开源日志框架 log4cplus 对日志进行格式化输出。已在生产上线完成，活跃用户数作为集团市场部的转售数据参考。 
2. 优化数据结构和数据处理流程，使话单处理速度提升了1倍多。
3. 负责新功能的添加和维护。

## 个人作品

1. 关系型数据库 mysql，oracle，postgre，kingbase 等 sql 上下文无关文法的实现及其语义分析，[项目地址](https://github.com/small-cat/myCode_repository/tree/master/antlr4_cpp)

2. mongodb api调用到mongodb shell cmd翻译器实现，[项目地址](https://github.com/small-cat/mongo_command_transfer)

   ```
   $ ./bin/mongodbcmd_transfer -f insert.txt 
   original ----->>>>>
   db.runCommand(   {      insert: "users",      documents: [         { _id: 2, user: "ijk123", status: "A" },         { _id: 3, user: "xyz123", status: "P" },         { _id: 4, user: "mop123", status: "P" }      ],      ordered: false,      writeConcern: { w: "majority", wtimeout: 5000 }   })
   
   translating ----->>>>>
   db.users.insert({[{_id:2, user:"ijk123", status:"A"}, {_id:3, user:"xyz123", status:"P"}, {_id:4, user:"mop123", status:"P"}], false, {w:"majority", wtimeout:5000}})
   ```

3. cbc 编译器前端实现，实现语法分析，抽象语法树的构建，变量消解，类型消解到静态类型转换，[项目地址](https://github.com/small-cat/cbc-cpp)

   ```
   源码部分：
   import stdio;
   
   int
   main(int argc, char **argv)
   {
       printf("Hello, World!\n");
       return 0;
   }
   
   编译输出抽象语法树：
   $ ./sesame -I test/import/ test/hello.cb --dump-ast
   variables:
   functions:
     FunctionName: main
     IsPrivate: false
     Parameters: 
       Params:
         Name: argc
         TypeNode: int
         Name: argv
         TypeNode: char**
     FunctionBody: 
       <<BlockNode>> (test/hello.cb, line 5, column 0)
       variables:
       statements:
         <<ExprStmtNode>> (test/hello.cb, line 6, column 4)
         expr: 
           <<FunctioncallNode>> (test/hello.cb, line 6, column 4)
           Expr: 
             <<VariableNode>> (test/hello.cb, line 6, column 4)
             name: printf
           Arguments:
             <<StringLiteralNode>> (test/hello.cb, line 6, column 11)
             Value: "Hello, World!\n"
         <<ReturnStmtNode>> (test/hello.cb, line 7, column 4)
         Expr: 
           <<IntegerLiteralNode>> (test/hello.cb, line 7, column 11)
           TypeNode: int
           Value: 0
   ```

   

## 个人评价
　　喜欢计算机科学与技术，希望能够在基础软件领域持续深耕。喜欢研究编译器技术，研究 llvm，阅读linux内核源代码，开设博客(blog.csdn.net/honglicu123, blog.wuzhenyu.com.cn)，与大家分享自己的学习心得和实践经验，希望能够与更多志同道合的朋友一起探讨，共同进步。
