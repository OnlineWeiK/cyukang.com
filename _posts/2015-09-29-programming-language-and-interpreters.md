---
layout: post
title: "Tiny Interpreters"
description: ""
category: 
tags: [Programming]
---

{% include JB/setup %}

After reading the first simple Scheme interpreter of bootstrap-scheme, I have some interests on studying various programming languages and interpreters. It's really fun to implement tiny programming languages. For learning a new programming language, a simple Scheme interpreter is a good starting project. Because in this small project we need to know some core aspects of a new programming language, including the basic I/O operation, abstraction method for expression representation, recursive for eval, and unit testing. Also mini Scheme is so easy for parsing, we can focus on data representation and eval.

Two programming language are best suited to implementing  interpreter, The first one is Scheme, which used in many famous PL books, such as EOPL, SICP, etc. Another good language in OCaml,
which have been a very hot spot in the history of programming language. It's very convenient to implement parser, and also because of the pattern matching and algebraic data types, it is nature for building AST and traverse on it.


For your references, I have these small projects during my studying of languages:

[rust-scm](https://github.com/chenyukang/rust-scm), which is a Scheme interpreter written in Rust

[GoScheme](https://github.com/chenyukang/GoScheme), yet another Scheme interpreter written in Go

[ocaml-scheme](https://github.com/chenyukang/ocaml-scheme), yet another Scheme interpreter written in OCaml

[eopl](https://github.com/chenyukang/eopl), hundreds of interpreters written in Scheme, according the EOPL exercises.

[toy-compilers](https://github.com/chenyukang/toy-compilers), still they are interpreters, but not compilers, with js_of_ocaml we can compile OCaml code to Javascript, then run it on web browsers!