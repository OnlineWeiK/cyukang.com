---
layout: post
title: "Types and Programming Languages (2)"
description: ""
category:
tags: [PL, Ocaml, Programming]
---
{% include JB/setup %}


## References

### Side effect
    In particular, besides just yielding results, evaluation of terms in these languages may assign to mutable variables (reference cells, arrays, mutable record fields, etc.), perform input and output to files, displays, or network connections, make non-local transfers of control via exceptions, jumps, or continuations, engage in inter-process synchronization and communication, and so on. In the literature on programming languages, such “side effects” of computation are more generally referred to as computational effects.

引用指向的对象可以是基本类型、组合类型，甚至是函数，把指向函数的ref放进对应的record，就变成一个简单的object，OOP的原型就出来了。


{% highlight ocaml %}
update = λa:NatArray. λm:Nat. λv:Nat.
     a := (λn:Nat. if equal m n then v else (!a) n);
{% endhighlight %}

通过这个习题的例子可以看出ref引进的副作用。

## Garbage Collection

### GC or not
    This is not just a question of taste in language design: it is extremely difficult to achieve type safety in the presence of an explicit deallocation operation.

     The reason for this is the familiar dangling reference problem: we allocate a cell holding a number, save a reference to it in some data structure, use it for a while, then deallocate it and allocate a new cell holding a boolean, possibly reusing the same storage. Now we can have two names for the same storage cell—one with type Ref Nat and the other with type Ref Bool.

### Pointer
    Pointer arithmetic is occasionally very useful (especially for implementing low-level components of run-time systems, such as garbage collectors), it cannot be tracked by most type systems: knowing that location n in the store contains a Float doesn’t tell us anything useful about the type of location n + 4. In C, pointer arithmetic is a notorious source of type safety violations.

Store typings: 引入引用后类型系统需要处理Cyclic reference structures，比如double linked list。Store typings就是一个locations到typings的映射。

实现fullref： 引用部分的实现非常简单，

{% highlight ocaml %}
  | TmRef(fi,t1) ->
      TyRef(typeof ctx t1)
  | TmLoc(fi,l) ->
      error fi "locations are not supposed to occur in source programs!"
  | TmDeref(fi,t1) ->
      (match simplifyty ctx (typeof ctx t1) with
          TyRef(tyT1) -> tyT1
        | TyBot -> TyBot
        | TySource(tyT1) -> tyT1
        | _ -> error fi "argument of ! is not a Ref or Source")
{% endhighlight %}
