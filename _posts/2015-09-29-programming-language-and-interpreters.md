---
layout: post
title: "Tiny Interpreters"
description: ""
category: 
tags: [Programming]
---

{% include JB/setup %}

After reading the first simple Scheme interpreter of [bootstrap-scheme](/2013/02/15/reading-bootstrap-scheme.html), I have some interests on studying various programming languages and interpreters. It's really fun to implement tiny programming languages. For learning a new programming language, a simple Scheme interpreter is a good starting project. Because in this small project we need to know some core aspects of a new programming language, including the basic I/O operations, abstraction methods for expression representation, recursive for eval, and unit testing. Also mini Scheme is so easy for parsing, we can focus on data representation and eval.

Two programming language are best suited to implementing  interpreter, The first one is Scheme, which used in many famous PL books, such as EOPL, SICP, etc. Another good language in OCaml,
which is a sweet spot in language design space: strict, type system and type-inferer, functional. It's very convenient to implement a parser, and also because of the pattern matching and algebraic data types, it is nature for building AST and traverse on it.

For your references, I have these small projects during my studying of languages(to be continued):

[eopl](https://github.com/chenyukang/eopl), hundreds of interpreters written in Scheme, trying to solve most of the EOPL exercises.

[rust-scm](https://github.com/chenyukang/rust-scm), which is a Scheme interpreter written in Rust

[GoScheme](https://github.com/chenyukang/GoScheme), yet another Scheme interpreter written in Go

[ocaml-scheme](https://github.com/chenyukang/ocaml-scheme), yet another Scheme interpreter written in OCaml

[toy-compilers](https://github.com/chenyukang/toy-compilers), still they are interpreters, but not compilers, with js_of_ocaml we can compile OCaml code to Javascript, then run it on web browsers!