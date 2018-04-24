+++
date = "2018-04-24T02:45:16+02:00"
draft = false
title = "GSoC 2018 Selection Results"
slug = "gsoc_2018_selection"
aliases = [
	"gsoc 2018 selection"
]
+++

We are happy to announce this year we accepted **five** students: two for [radare2](http://rada.re) itself, two for [radeco]( https://github.com/radare/radeco ) and one for [cutter](https://github.com/radareorg/cutter).

## HMPerson1

Hi, I'm Michael Zhang, also known as [HMPerson1](https://github.com/HMPerson1). I'm a first-year student at [Purdue University](https://purdue.edu/). I use radare2 regularly when playing CTFs to disassemble and analyze binaries. Although radare is very powerful, there's only so much that can be done staring at dissassembly. Having access to source code would make analysis much easier.

This summer, I'll be working on adding structured control flow recovery to [radeco-lib](https://github.com/radare/radeco-lib), a radare2-based decompilation library. Currently, radeco-lib can create an IR in SSA form from [ESIL](https://radare.gitbooks.io/radare2book/content/disassembling/esil.html) produced by radare2. The instructions in this IR are organized into [basic blocks](https://en.wikipedia.org/wiki/Basic_block), and the flow of execution between these blocks can be described by a [control flow graph](https://en.wikipedia.org/wiki/Control_flow_graph).

My project is to convert the control flow graph described by this IR into a more structured form, specifically into the control flow structures used in C (i.e. `for` and `while` loops; and `if-else` and `switch` statements). To do this, I will be implementing the algorithm described in the paper titled [No More Gotos](https://doi.org/10.14722/ndss.2015.23185).

Kriw and I will be working together to create a working decompiler by the end of this summer that can correctly decompile most binary programs into human-readable source code.

## Kriw

Hi, I'm [kriw](https://github.com/kriw). I'm a student at Tokyo Institute of Technology. I usually use radare2 to analyse some programs statically (mainly for CTF) and I need a decompiler of multi-architecture for fast and better analysis. I have contributed to `radare2`/`radeco-lib` a little since last December. My previous contributions were in `rax2`, and on improving the analysis of radeco-lib.

For GSoc'18, I'll be working on developing radeco-lib together with HMPerson1 to make the decompiler produce C code. Our major goal is to produce C-AST and pseudo-C code. The current status of radeco-lib is what HMPerson1 has written above. I'll do my best to implement a converter to C-AST from the result of analysis and finally, output pseudo-C code.

Our additional objectives are to improve various analysis heuristics of the decompiler, and to integrate with radare2 so that everyone can easily use it.

## Cyanpencil 

Hello, I'm Luca, also known as [cyanpencil](https://github.com/cyanpencil). I am an italian CS student in my final undergraduate year.
I got to know radare during my first CTF competition and I immediately fell in love with it. In particular, I was fascinated by its aesthethics and integration with the console. For this reason, in this Summer of Code I'll work on console interface improvements. 

In detail, the first part of my GSoC will be devoted to fixing UTF-8 related bugs and implementation of an API to generate ASCII art tables easily. The second part will be focused on the graph view, with the addition of features like cursor mode and node highlighting (like in IDA). Finally, the last part will revolve around writing the tests and documentation (r2book) and adding an autocompletion widget similar to vim:

![example screenshot](https://camo.githubusercontent.com/3b2eb4c4dc72b975032f536a93260dfe7353c23f/687474703a2f2f6e6f736d696c65666163652e72752f696d616765732f676f636f64652d73637265656e73686f742e706e67)

Even if I already sent some bugfixes (example: [1](https://github.com/radare/radare2/pull/9685), [2](https://github.com/radare/radare2/pull/9607), [3](https://github.com/radare/radare2/pull/9591)), I'm really looking forward to contribute seriously to this awesome tool.

## sivaramaaa

Hi, I’m [Siva Rama Krishnan](https://github.com/sivaramaaa), a third year undergraduate student from India. I  have contributed several small patches to r2 over the last few months, primarily fixing bugs in analysis. 

Currently radare2 support C-like types with `t` command, we could parse the C type definition from C header, or load from "precompiled" SDB file. 

The major goal of this task is to implement an analysis that would infer types of variables using known libc function signature. The core goals for this summer also include : 

* Improve and test the current variable recovery mechanism
* Improve and test `t` commands
* Ability to convert numeric immediates (and offsets) into structures offsets/global offsets in disassembly
* Handling of complex nested structures
* Type inference engine based on function arguments types, return types
* Export and import return and argument types with function signatures



## mandlebro

Hi! I'm Filipe also known as [mandlebro](https://github.com/fcasal) and I am currently doing my PhD at [Técnico ULisboa](https://tecnico.ulisboa.pt). I have been using radare2 for some time to do reversing CTF tasks and AVR emulation, as well as making contributions to both [r2](https://github.com/radare/radare2/pulls?q=is%3Apr+author%3Afcasal+is%3Aclosed) and [Cutter](https://github.com/radareorg/cutter/pulls?q=is%3Apr+author%3Afcasal+is%3Aclosed).

This summer I will be working on [Cutter](https://github.com/radareorg/cutter), the GUI for radare2. Specifically, I will be integrating the emulation and debugging capabilities of radare2 into Cutter. This will allow new radare2 users or users which do not like the CLI to take advantage of the powerful debugging/emulation features radare2 has to offer.


