---
layout: post
title: "Writing a Compiler from Scratch in Rust Part 1: The Lexer"
---

# Introduction
In May of this year I started embarking on a project to write a full compiler for the [C0](http://c0.typesafety.net/tutorial/) programming language. This has been something I've been wanting to do for a while, but I had to put development of this compiler on the backburner due to university and work commitments. As of September I have picked up production of the compiler again.

[Source repo](https://github.com/nashpotato/C0-Compiler)

# What are Compilers?
For a brief primer, compilers are one of the most interesting but overlooked and underappreciated areas of CS. A compiler, as most programmers know, is involved in transforming a high level programming languages source code into something that can be understood and executed by a computer. The theory behind how these works is an active research area, and has been for decades. However, apart from compiler engineers themselves, most coders don't really get into how compilers really work. For me personally I have always been interested in compilers, but never felt too confident in my ability to develop them. But that's changed.

# The Compiler
Compilers are typically split into several stages. Each stage represents a different part of the compilation stage. The diagram below illustrates these different stages.

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
    Token::LBracket,
    Token::RBracket,
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

What can be seen is that we're essentially from the source language and just generating representations for key components. Notice that whitespace is ignored as for a language like C0, whitespace serves no purpose other than to make it easier to read for people. 
The reason that these tokens are encapsulated in a vector is that this is a dynamic data structure, and the data we have is ordered.

# Conclusion
Logically this isn't hard to understand, we have just provided a representation for the entire source file by just linearly reading the source thereby giving meaning to our source. This is what makes lexers so simple. So simple in fact that some compilers completely do away with an explicit lexing step, and just parse the language straight from source. This does in a sense make this blog post redundant, and I guess it serves a greater purpose as introducing this blog series.

Now that our source has an attached "meaning", we can start parsing over the source language which will be covered in part 2. The source for the lexer can be found [here](https://github.com/nashpotato/C0-Compiler/blob/master/src/lexer/lexer.rs).
 
