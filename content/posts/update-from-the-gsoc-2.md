+++
date = "2015-06-18T14:55:42+02:00"
draft = false
title = "Update from the GSoC 2"
slug = "update-from-the-gsoc-2"
aliases = [
	"update-from-the-gsoc-2"
]
+++
As part of GSoC I (dkreuter) and sushant94 have been working the last three weeks on what should become the basis for a decompiler integrated with the radare2 reversing framework. 

For now it's a standalone program written in Rust that can read the radare2 code format ESIL. The rough process involves generating control and data flow graphs in [SSA form](http://pp.ipd.kit.edu/firm/GraphSnippets "Examples for SSA graphs in an unrelated project") for the input, applying simplifications on that, similar to compilers, and picking appropriate constructs in a target language to represent the input. The result will be a more intuitive representaton of the analyzed program.

The task is hard even in theory, as a program that prints `4` could've been compiled from `print(4)` or `print(2+2)`. There's no way to know. Right now however, we're just trying to get the first and simplest case (4→4) to work. But the insight, that the decompilation process is neccessarily a interpretation process, is what I try to consider in my designs.

The alternatives to Rust we considered were OCaml and C++. None of us has written Rust or OCaml before, but seemed like many C++ skills would be transferrable to Rust (at some cost of idiomaticness). The first two weeks were full of gotchas, but now I'm very comfortable with the language. The Rust IRC channel was very helpful in that regard.
Rust has plenty of cool features including a very expressive typesystem (`X<U> extends Y<Z> + Q where Z extends X<U>`, etc.), a checker that ensures there are no double frees, aliasing non-const pointers or dangling pointers (without garbage collector), tagged unions, a nice build system unlike C++ (Apparently the C++ modules proposal didn't make it for C++17), type checks for metaprogramming (C++ uses duck typing instead), nicey integrated documentation generation and testing and pattern matching.
In C++ terms, all references in Rust are `const restrict * const` and have move-semantics per default. It does make some tasks more tedious than normally. (eg. `swap(&x[4], &x[5])` needs workarounds to compile) But I still think that these defaults will prevent more problems that they cause.

So while sushant94 has been working on parsing and representing the data coming from radare2 (with [good results](/update-from-the-gsoc/) it seems), I've been working on the graph data structures, which turned out to be more complicated than anticipated.
Firstly, an SSA graph is doesn't only have nodes and edges, it also has one level of nesting of nodes (computations in basic-blocks) and it also has "edge-order" (A node representing subtraction needs to know which edge represents its first or second operand.) meaning that we couldn't just use a preexisting library (without adaptions). Also, the fact that Rust wants a statically determinable tree of ownership to exist clashes a bit with the requirements of a graph.
In the end we used an existing graph library for the upper level (the basic blocks) and manually manage the lower one with instruction lists in each block. Integers are used as "pointers" between nodes on both levels. Another challenge were the Phi nodes which (unlike other instructions) have a variable number of operands. They have as many operands as their containing basic block has incoming control flow edges. It leads to a lot of special casing, making the code messy. I hope to find time to revisit this later (after GSoC probably).

Once that's done we'll work on some more integration with radare2 and begin the first code that interacts with the graphs, like simplification (`2+x+2 → 4+x`) and dead code elimination (`if(0){/*delete this*/}`).