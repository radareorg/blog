+++
date = "2014-05-06T15:14:16+02:00"
draft = false
title = "x86 Capstone tests"
slug = "testing-coverity"
aliases = [
	"testing-coverity"
]
+++
As you may know, we are using the [capstone]( http://capstone-engine.org/ ) as a disassembling engine for several architectures. We are even planning to use it as main engine and to ditch [udis86]( https://github.com/vmt/udis86 ).
Since the x86 is one of the most common architecture, we want to be sure that the transition does't break anything.

This is why our resident test writer *maijin* did the following things:

 - He added **one thousand** x86-related tests!
 - Every test is now a one-liner, thanks to *l0gic*'s refactoring.
 - New bugs were detected, and are now beeing fixed either by radare2,or directly by the Capstone team
 
We would like to thank [corkami]( http://corkami.googlecode.com/svn-history/r79/trunk/misc/opcodes32.asm ) because we borrowed some of his work, and of course the capstone team.
## How to run those tests?
Just type `make tests`; this will:

 1. Clone or update the [r2-regressions]( https://github.com/radare/radare2-regressions ) repo.
 2. Launch the whole testsuite.
You can also launch only the capstone one by typing `make capstone` inside the r2-regressions folder, or `make asm` for every asm ones.
