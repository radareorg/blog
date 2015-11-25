+++
date = "2015-11-25T12:47:13+02:00"
draft = false
title = "Analysis By Default"
slug = "analysis-by-default"
aliases = [
	"analysis-by-default"
]
+++
Analysis By Default
===================

Many people that starts using radare2 complains about having a different workflow than other similar tools like IDA or Hopper.

Probably the most anoying part for them is not to run the analysis at start. And this is the reason why I'm writing this blog post right now.. to avoid having to explain this thing again and again :)

To begin with, r2 is a pretty broad tool. It didn't even started as a disassembler in mind, so for example, code analysis doesn't makes sense to be done when you load a hard disk image or a memory dump for example. But some others will complain that we can just run the analysis when the file format is known to contain code.

But then, several other reasons appear in the scene. Let's enumerate them:

Slow process
------------

Code analysis is not a quick operation, and not even predictible or takes a linear time to be processed. This makes starting times pretty heavy, compared to just loading the headers and strings information that is done by default.

People that is used to IDA or Hopper, they just load the binary, go out to make a coffee and then when the analysis is done they start doing the manual analysis to understand what the program is doing. It's true that those tools perform the analysis in background, and the GUI is not blocked. But this takes a lot of CPU time, and r2 aims to run in many more platforms than just high-end desktop computers.

Radare2 can also load the binary information in background, and perform there's some initial work to do code analysis in async and threaded mode, but this is just not an excuse for running a heavy operation by default on start.

Perfection
----------

In the world of static code analysis tools, perfection doesn't exist. This is somehow a pretty complex topic that depends on the target architecture, file format, 

In our case. radare2 is able to load many more file formats than any other tool in the market and as long as fuzzing is part of our development model, we are able to load pretty messed up files where others just crash in a flawlessly way. This turns to create some side effects in the code analysis, by forcing it to walk into strange and invalid code paths when it is never going to be executed in a real environment.

Also, r2 have a single analysis engine (which can be changed or extended by plugins), this contrasts with other tools that just have a specific dedicated engine for each architecture, which is probably better for very specific use cases, but probably not good when you are out of the scope of their suposed use cases.

Also, bear in mind that switch tables are not always easy to resolve statically, most tools just expect those constructions to be standard building blocks spitted by compilers, but it can be different. Many times, you will always need some manual interaction with the tool to get full and proper analysis of the binary.

Configuration
-------------

As described in the previous section, analysis is not an exact science, so it needs some manual tuning in order to optimize the results depending on the target. Yeah, I know, malware is probably one of the main sources that requires this kind of options to be defined.

In r2, we have 26 different `anal.` configuration variables that can be used to change the behaviour of the analysis engine. Probably the most interesting ones are the following:

* anal.afterjmp
* anal.depth
* anal.eobjmp
* anal.esil
* anal.hasnext
* anal.nopskip

See `e??anal.` to get some more detailed descriptions for them.

There's even two more handy configuration variables (anal.from and anal.to) that allows you to restrict the boundaries of the analysis. This way you can for example focus on analyzing only the application code in a statically compiled binary by excluding the library code, which is usually at the end of the program, this can be easily detected by manually reading the disassembly.

Types
-----

Running the analysis in radare2, is not just a single step or action. We have many different commands that perform different kind of analysis and let you identify new functions and references in faster or funky ways, which are handier in different situations. Let's just enumerate some of them:

* Find functions by prelude instructions (aap)
* Identify functions by following calls (aac)
* Detect jump tables and pointers to code section (/V)
* Analyze opcode absolute and relative references (aar)
* Find code/data/string references to a specific address (/r)
* Emulate code to identify new pointer references (aae)
* Use binary header information to find public functions (aas)
* Assume functions are consecutive (aat)
* ...

radare2 is not a click-and-run program, it's a set of orthogonal tools and commands that allows you to understand, analyze, manipulate and play with a large list of binary types. And, as long as every single point in the above list will take some time, you will probably not want to run them all in a shot, also, the execution order will also alter the results. Only experience and understanding will give you control on what you are doing.


Use Cases
---------

Not everytime you load a binary you need or want an entire code analysis of it. Let's say, for example, that you have already performed a previous analysis and you have a project database stored somewhere and you just want to load it into this binary.

Radare2 allows you to do this, but not only this, because, as long as projects are text files, you can rebase all the symbols in case the binary shifted the symbols to a new address, etc..

Also, it supports signatures, so you can just find function entrypoints by loading a signatures files which can be generated by radare2 or IDA (FLIRT). In this case you will probably not need to analyze the entire binary to understand what it is doing.

If you are going to do some automated process in a binary you want to that as fast as possible, by loading the only parts of the binary you want and doing only the steps you need to perform your actions.

In other words: 90% of the time you are going to analyze a binary will be to to find references to some imports or strings, read the code of the function, maybe analyze 2 or 3 functions and go away. You don't need full code analysis for forensics, or exploiting, or binary patching, and for reverse engineering, you'll probably focus on one or two specific problems which doesn't require a complete program analysis, neither for extracting information of a binary.

The only reasons for doing a full code analysis are similar to the ones for decompilation, which is to recover lost source code, transpile the program for another arcuitecture or operating system, rebasing it, etc. I would say that those cases are less than 10% or less of what most people needs.

Final Words
-----------

If you are used to this old-fashioned, lazy workflow, you will probably not going to switch to radare2 as your main tool.

But you can just load r2 with the `-A` flag to get some default analysis done at start time. But again, this will be probably a loss of time if you are an experienced reverse engineer.

We are not closing the door to perform a full analysis of the binary, but I just think that it's something that shouldn't be done by default and at start.

We enforce users to think in their workflows in order to better understand the problem they are facing and solve it in an optimal way, saving cpu, memory and why not: cats.


Have phun!

--pancake
