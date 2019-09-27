---
layout: post
title: "Writing a Compiler from Scratch in Rust: The Lexer"
---

# Introduction
In May of this year I started embarking on a project to write a full compiler for the [C0](http://c0.typesafety.net/tutorial/) programming language. This has been something I've been wanting to do for a while, but I had to put development of this compiler on the backburner due to university and work commitments. As of September I have picked up production of the compiler again.

[Source repo](https://github.com/nashpotato/C0-Compiler)

# What are Compilers?
For a brief primer, compilers are one of the most interesting but overlooked and underappreciated areas of CS. A compiler, as most programmers know, is involved in transforming a high level programming languages source code into something that can be understood and executed by a computer. The theory behind how these works is an active research area, and has been for decades. However, apart from compiler engineers themselves, most coders don't really get into how compilers really work. For me personally I have always been interested in compilers, but never felt too confident in my ability to develop them. But that's changed.

# Why Rust?
[Rust](https://www.rust-lang.org/) is a increasingly popular systems programming language. The reason for this popularity is I think down to one reason really.

**For a lot of problems it's preferable to C/C++**: C/C++ are by far the most commonly used languages when it comes to systems programming but as a language it is quite old. Rust offers a host of new paradigms within the space of programming language design that have allowed for programmers to write fast, safe,high-performant code. Language features such as immutability, the ownership model and it's rich type system mean that programming with Rust is not only more enjoyable but just safer. The Rust compiler, for any Rust programmer, is by far your best friend and the community and documentation are incredibly helpful. That is not to say that Rust is without flaw. Compared to C/C++, Rust isn't necessarily the language of choice for very low-level close to the metal programming, and of course the ecosystem of systems development is still heavily dominated by C/C++, I still believe Rust to be a great language for development, and is certainly my choice for building this compiler.

# The Compiler
And now I am done with my diatribe, now I can talk about the compiler. Compilers are typically split into several stages. Each stage represents a different part of the compilation stage. The diagram below illustrates these different stages.

![Compiler Stages](/assets/compiler-phases.jpg)

Each stage does the following

- **Lexical analysis**: in this stage we take the source code from the original program and extract out tokens which represent different lexical components. 
- **Syntax analysis**: in this step we build a representation of syntactic meaning called an [abstract syntax tree](https://en.wikipedia.org/wiki/Abstract_syntax_tree).
- **Semantic analysis**: here we perform analysis on the AST
- **Intermediate code generation**: here we generate something called intermediate representation code. The purpose of this step is to generate a platform independent representation of the language that can be passed for optimisation. This eliminates the need for writing optimisers that are platform dependent. So if we are compiling for an ARM processor, or a x86 processor, the IR step ensures that we optimise for a representation that is independent from these architectures before being compiled for these architectures. This is easier than writing optimisers for individual architecture languages.
- **Optimisation**: this is the really fun step where code is optimised to improve performance. [There's a wide range ](https://en.wikipedia.org/wiki/Optimizing_compiler#Types_of_optimization) of optimisations that can be performed in this step. 
- **Code generation**: here we generate the code that we are compiling to.
- **Target code optimisation**: this step may be useful if there are optimisations that are dependent on the target code language.
- **Target code generation**: here we produce the executable code for the target.

The descriptions above are quite high level but cover the gist of what is going on in compilers. The purpose of this post is to discuss the lexing step which I have implemented in my compiler.

# The Lexer
The lexer is a by far the simplest stage in the compilation step and really just involves taking the original source code, and producing tokens that represent individual language components. This is quite hard to explain in words, so let's look at some code. Let's look at an example C0 program

```c

int main(){
    int x = 0;

    return x;
}
```

and let's look at the output of passing this program through the lexer

```rust
vec![
    Token::Int,
    Token::Undefined(Some('m')),
    Token::Undefined(Some('a')),
    Token::Undefined(Some('i')),
    Token::UndefineD(Some('n')),
    Token::LCurly,
    Token::Int,
    Token::Undefined(Some('x')),
    Token::Eq, 
    Token::Num(0),
    Token::SemicColon,
    Token::Return,
    Token::Undefined(Some('x')),
    Token::SemiColon,
    Token::RCurly,
    ]
```