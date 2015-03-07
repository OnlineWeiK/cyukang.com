---
layout: post
title: "Types and Programming Languages (1)"
description: ""
category:
tags: [PL, Ocaml, Programming]
---
{% include JB/setup %}

最近掉进另外一个PL的坑里面，就是想读一下[这本书](http://www.cis.upenn.edu/~bcpierce/tapl/)，顺便继续熟悉一下Ocaml。下面的记录是阅读过程中的一些摘录和理解。

1-2章是数学预备部分，理论部分有些地方比较难懂，主要是一些数学符号看久了眼花。
解释器的实现大多只用看syntax.ml和core.ml，就是语法和具体eval，typeof函数。

## Untyped Systems
arith是一个无类型的解释器，是后面所有章节的基础。printtm_Term用了Format模块来格式化打印。

## The Untyped Lambda-Calculus
浅显易懂的Lambda-Calculus解释，同时列举了一些lambda calculus扩展其他语言部分的例子。

## An ML Implementation of the Lambda-Calculus

shifting和substitution的实现挺难看懂的，本质上是把context里面的变量用index来替换，处理变量查找的一种实现而已。eval部分是非常地简洁，我觉得ML系的语法看起来比Scheme都舒服紧凑。

    Just because you’ve implemented something doesn’t mean you understand it.
                                                        —Brian Cantwell Smith

说起来全是泪，用这种函数式的编程语言来解释自己确实比较简单，但现实往往不是这样。语言能比较容易地实现自己至少可以表明语言的内核挺小，一个语言能实现bootstrap是成熟的一个表现。Rust的实现最初是用Ocaml写的，然后编译出一个Rust的编译器，然后用上一版本的Rust再重新实现Rust编译器。

## Typed Arithmetic Expressions

tyarith是最简单的带类型的解释器，有bool和Nat类型。

    Progress: A well-typed term is not stuck (either it is a value or it can take a step according to the evaluation rules).
    Preservation: If a well-typed term takes a step of evaluation, then the resulting term is also well typed

These properties together tell us that a well-typed term can never reach a stuck state during evaluation.

Safety = Progress + Preservation

## Simply Typed Lambda-Calculus

     In general, languages in which type annotations in terms are used to help guide the typechecker are called explicitly typed. Languages in which we ask the typechecker to infer or reconstruct this information are called implicitly typed.

     Well-typed programs cannot “go wrong.” —Robin Milner (1978)


## An ML Implementation of Simple Types
simplebool是一个只有bool类型的解释器，但是加上了函数。typeof挺简单，主要是函数这里注意处理形参和实参:

{% highlight ocaml %}
  | TmAbs(fi,x,tyT1,t2) ->
      let ctx' = addbinding ctx x (VarBind(tyT1)) in
      let tyT2 = typeof ctx' t2 in
      TyArr(tyT1, tyT2)
  | TmApp(fi,t1,t2) ->
      let tyT1 = typeof ctx t1 in
      let tyT2 = typeof ctx t2 in
      (match tyT1 with
          TyArr(tyT11, tyT12) ->
            if (=) tyT2 tyT11 then tyT12
            else error fi "parameter type mismatch"
        | _ -> error fi "arrow type expected")
{% endhighlight %}

if的判断部分必须为bool，而且两个分支必须为同一类型:

{% highlight ocaml %}
  | TmIf(fi,t1,t2,t3) ->
     if (=) (typeof ctx t1) TyBool then
       let tyT2 = typeof ctx t2 in
       if (=) tyT2 (typeof ctx t3) then tyT2
       else error fi "arms of conditional have different types"
       else error fi "guard of conditional not a boolean"
{% endhighlight %}

## Simple Extensions
在上一章的基础上，加上各种Drived Form。

####Sequencing
是多个表达式串，这在有副作用的语言里面很常见。另外也可以把t1;t2理解为(λx:Unit.t2) t1。

####Wildcards
如何翻译好，意思就是无用形参可以不指定名字。

####Ascription
是指类型缩写(或者昵名)，C++里面的typedef，和Rust里面的usize as U都是。这个的好处在于文档和接口更清晰，如果函数的参数可以是函数，类型加进以后语法看起来就比较繁琐了，用类型缩写更清晰。typechecker的时候当然需要展开来进行。ascription和casting也有一定关系。

增加各种简单的基础类型，比如String，还有Pairs，Tuple，Record， Sum，Enum，List。支持一种类型除了一个新类型名字外，其evaluation rules和type rules也要明确。这里的datatypes是按照Ocaml的语法来说明的。

因为加上了好多种类型，fullsimple这个解释器复杂多了。

####Type Dynamic

     Even in statically typed languages, there is often the need to deal with data whose type cannot be determined at compile time. This occurs in particular when the lifetime of the data spans multiple machines or many runs of the compiler—when, for example, the data is stored in an external file system or database, or communicated across a network. To handle such situations safely, many languages offer facilities for inspecting the types of values at run time.

####General Recursion
typed lambda-calculus加上fix combinator就是一门极小的但是是full abstraction的语言。Ocaml里面的letrec可以用来定义递归函数。fix point的概念需要继续理解。
