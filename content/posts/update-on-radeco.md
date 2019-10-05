+++
date = "2015-09-26T23:51:06+02:00"
draft = false
title = "Update On Radeco"
slug = "update-on-radeco"
aliases = [
	"update-on-radeco"
]
+++
This post is to outline the [work](https://www.google-melange.com/gsoc/project/details/google/gsoc2015/sushant94/5733935958982656) completed during the Google Summer of Code 2015 (GSoC) period and show you a glimpse of radeco and where we are heading with it.

For those who are not aware, radeco is a decompiler framework that is developed and maintained by the radare team. The entire framework is open source, flexible and reusable. The base of radeco is radeco-lib that implements the analysis and transformations that are needed for decompiler. radeco uses radeco-lib to perform decompiler. You can checkout these repositories [here](https://github.com/radare/radeco) and [here](https://github.com/radare/radeco-lib)

## Features
This section gives a brief overview of the internal working of radeco as of today. These may change over time as more features and analysis are added. Nonetheless this does form the basic foundations on which radeco is built.

Radare2 uses an internal IR, ESIL (Evaluable Strings Intermediate Language) for its emulation and analysis needs. Since radeco is built on top of radare2 it parses ESIL to an internal representation. The first step of building the decompiler framework is to develop a good intermediate language. During the start of GSoC period we investigated several intermediate languages(IL) like REIL, RREIL, Vine IL and BAP IL. We tried to incorporate the best features from all these ILs while trying to keep the language as simple as possible. The main reason ESIL is converted into another IL is that ESIL is not really suited for static analysis, it was built for emulation and dynamic analysis. Since there are already a lot of architectures that are supported by ESIL, it makes more sense to lift ESIL into our IL rather than writing a lifter for every architecture again. Initially, ESIL from radare2 is converted into a text based IL which is then transformed into a tree-like SSA representation that is used for all further analysis.

The second step is to transform ESIL that is emitted by radare2 into the IL. This is done in two stages. The first step is a basic transformation that transforms ESIL into radeco IL. This is then broken into basic blocks and a Control Flow Graph (CFG) is constructed. The Control Flow information is then used to transform the IL into a tree like SSA representation. This is used for further analysis and optimisations. Radeco supports a few analysis like constant propagation and dominator tree construction (this feature is not exposed using the API yet). Apart from analysis radeco also performs a basic level of dead code removal to declutter the code and remove unnecessary operations.

Currently, radeco can output it's internal representation to [dot file](https://en.wikipedia.org/wiki/DOT_(graph_description_language)) which can be converted into png/svg using utilities like graphviz. While this may not seem very useful, they do form the base of the entire decompiler and identifying bugs at this stage is important. Much of the work during the GSoC was laying the foundations and deciding an architecture that would allow the library to be as flexible as possible. Now that we have a solid foundation and a good architecture in place we can move at a faster pace and implement all the analysis needed to build a decompiler.

## Usage
Radeco makes it easy for users to write their own custom analysis while leveraging the analysis that have already been implemented in radeco. One of the fundamental principles of radeco is allowing users to interact with the output produced from every stage of radeco's analysis, make corrections or changes where necessary and then continue with the analysis. To make this happen radeco-lib introduces `Pipeline`. This allows users to decide which stages of the analysis is to be run, the order in which they are to be run and allow them to tap into the output that is generated after every stage. For more information about `Pipeline` in specific, check out the [pipeline](https://github.com/radare/radeco#pipeline) section of the README.

Radeco is intended to be used from within r2. Good news is radeco
is now integrated with r2. Here is a quick walkthrough on the various options that you maybe required to use with radeco. For example:

```
> #!pipe <path/to/radeco> -p r2,esil,cfg,ssa,const,dce,svg`
```
By default radeco takes the current offset from r2 as the offset to begin decompilation from. The `-p` option specifies the order and the stages of the pipeline radeco should run. In the above example, radeco is asked to run the following stages in order:

* Read ESIL from r2
* Parse ESIL into radeco IL
* Construct CFG
* Construct the SSA
* Run Constant Propagation
* Run dead code elimination
* Create an svg for the dumped dot files
 
By default, radeco dumps all the output from every stage to the output directory. To suppress output from a particular stage, prefix the stage with a '-' as in:

```
> #!pipe <path/to/radeco> -p r2,esil,-cfg,ssa,const,dce,svg
```

For more detailed information about radeco and all its options, please check `radeco --help`.

Below is an asciinema showing the example usage of radeco:
[![asciicast](https://asciinema.org/a/cjou29227jmah5fgvw2l8hkx5.png)](https://asciinema.org/a/cjou29227jmah5fgvw2l8hkx5)

A pre-built version of radeco is to be released soon, but for now you will need a [rust](https://www.rust-lang.org/) compiler (preferably nightly) in order to compile and try out radeco.

And oh! We also have (some) documentation for [radeco](http://radare.github.io/radeco/radeco/index.html) and [radeco-lib](http://radare.github.io/radeco-lib/radeco_lib/index.html).

## Challenges
One of the main challenges we faced during the project phase was developing the internal representation for storing the SSA form. ESIL has features that are not present in other intermediate languages that make this process slightly tricky (this is also why ESIL is suited for emulation unlike other languages). ESIL has a concept of internal variables. These are basically variables that the ESIL VM maintains and hold a special meaning to the VM. All internal variables in ESIL are prefixed by `$`. The main use case for these variables are for computation of processor flags. To be able to compute the flags on-the-fly when needed the ESIL VM also holds three other values: 

* The value after the last operation that modifies or sets a flag (`cur`),
* The value before the last instruction that modified the flag (`old`) and
* The size of the last operand(s) (`lastsz`).

For example, let us consider the x86 zero flag. In ESIL, this is represented by `$z`. The VM on encountering a `$z` during its execution will perform the actual computations needed to decide the value of the flag, which in this case is checking if `cur == 0`. This means that the parser for radeco must also keep track of these values (`cur`, `old` and `lastsz`) and substitute the correct set of operations for each flag when it encounters an internal variable.

The other challenge that we faced was an architectural one. One of the principles that we kept in mind while developing radeco is reusability. We wanted all the analysis implemented in radeco to be reusable. This allows users to implement their own data structures for storage while still being able to use the analysis that radeco provides. To be able to achieve this we leverage the power of [rust traits](https://doc.rust-lang.org/book/traits.html). Almost everything inside radeco is implemented in terms of traits to allow multiple implementations and reusability. Currently it is mandatory to use the radeco IL instruction set (while having the freedom to store the data in any manner) to be able to use the analysis that radeco provides. In the future, we aim to make this generic too, allowing users to use their own ILs and instruction sets. The main tradeoff for us during the summer was flexibility vs. time. The more flexible we needed the framework to be, the more time we needed to implement features in a way that would allow for this. Hence we decided to restrict ourselves and expand to support custom ILs later.

## Future steps
Radeco is under heavy development and has a long way to go before being a full decompiler framework. Here is a quick and incomplete list of what to expect from radeco in the near future:

* Type Inference
* Type Propagation
* More analysis
* A full C-Writer module to output readable, C-like pseudo code
* Control flow restructuring and reconstruction
* Full integration with radare2
* Text form for the internal IR allowing other tools to interact and use radeco for analysis
* Maybe even a GUI for radeco!

## Preview
Below is a small example to show constant propagation as implemented in radeco right now.

ASM of input binary:
```asm
global _main
section .text
    main:
        mov rax, 2048
	cmp rax, 2048
	je  equal
	add rax, 1
	equal:
	mov rbx, rax
	ret
```
It is clear from the input ASM that the jump is always executed and the instruction right after the jump is never executed. This aim is to check if radeco can infer this using constant propagation. This ASM was hand written as no optimizing compiler would produce this code (as they perform constant
propagation too).

The binary generated is disassembled by radare2. Below is the ESIL output that radare2 generates and provides radeco with:
```
384:	2048,rax,=
394:	2048,rax,==,%z,zf,=,%b64,cf,=,%p,pf,=,%s,sf,=
400:	zf,?{,408,rip,=,}
402:	1,rax,+=
408:	rax,rbx,=
411:	rsp,[8],rip,=,8,rsp,+=
```
__NOTE__: Due to the large size of images, I've chosen not to embed the full images here. Please feel free to checkout the links for outputs.

This ESIL is parsed by radeco into an intermediate radeco IL: [Non-SSA IL](https://raw.githubusercontent.com/sushant94/sushant94.github.io/new_post/blog/images/cfg_2.png)

The CFG is then transformed into the SSA representation. The first block shown below for reference:
![ssa](https://raw.githubusercontent.com/sushant94/sushant94.github.io/master/blog/images/ssa_small.png)

[Link](https://raw.githubusercontent.com/sushant94/sushant94.github.io/new_post/blog/images/ssa.png) to full image

Constant propagation is run on the SSA form. Radeco implements [Sparse Conditional Constant Propagation](https://en.wikipedia.org/wiki/Sparse_conditional_constant_propagation) which performs constant propagation and unreachable dead code elimination simultaneously.

The first block is shown below for reference.

![after constant propagation](https://raw.githubusercontent.com/sushant94/sushant94.github.io/master/blog/images/const_small.png)

[Link](https://raw.githubusercontent.com/sushant94/sushant94.github.io/new_post/blog/images/const_prop.png) to full image

As you can see from the above output, the unreachable branch is eliminated and radeco infers correctly the values of registers and flags after the function has been run.


## Contributing
We encourage you to give radeco a try. Contribution in the form of bug reports, pull requests, improving documentation or even just community suggestions are welcome. If there is any feature that you miss and you would like to see file a report with a description of the same so that we can address this. Providing a backtrace and the sample binary (where possible) would help us track down the bug much faster. Please consider this while filing a report.

If you're interested in using radeco for your own tools and have some questions, feel free to join #radare on Freenode. We would be more than happy to help you out!

--------
This is a repost from: http://sushant94.me/2015/09/21/radeco/
