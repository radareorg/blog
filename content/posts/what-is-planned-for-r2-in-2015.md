+++
date = "2015-01-27T19:39:56+01:00"
draft = false
title = "What is planned for r2 in 2015?"
slug = "what-is-planned-for-r2-in-2015"
aliases = [
	"what-is-planned-for-r2-in-2015"
]
+++
That's an interesting question, isn't it?

Our [last release]( http://radare.today/radare-0-9-8/ ) was intended to be focused on bug-fixing, but we *accidentally* added tons of features, a new webui, enhaced debugger, code emulation and much more. This was supposed to land in the [0.9.9]( https://github.com/radare/radare2/milestones/0.9.9 ) version, which will be ready in February.

Local variable detection has been uncommented and it's now using [SDB]( https://github.com/radare/sdb ) as storage and supports basic X86-32/64 and ARM constructions.

Currently, you can do several low level analysis operations like manually define/resize/remove/merge/... functions (ask `a?` about this), or find function preludes in new ways (using the `ac` command). There are also different (selectable) analysis loops.

The current [ELF]( https://en.wikipedia.org/wiki/Executable_and_Linkable_Format ) parser is ~~awful~~ and a bit suboptimal, but fortunately, [TheLemonMan]( https://github.com/LemonBoy ) is working on a simplified version. There are only a few tests to fix before getting it merged in master!

But before that it will need to be fuzzed to avoid unexpected crashes (we have already fixed several crashes thanks to zzuf, radamsa, nightmare, afl, asan and other fuzzing helpers in r2).

In addition pancake added support to force a specific RBin plugin to be used when parsing a new RIO file, this allows anyone to replace the stock ELF parser without having to recompile the whole lib.

Pancake and condret are working on [ESIL]( https://github.com/radare/radare2/wiki/ESIL ), and you can already emulate entire chunks of code in radare2! No need to setup a whole exotic QEMU environment to get the result of a single function!

Also, the upcoming analysis engine will rely on the aforementioned ESIL, allowing the get jump destinations, compute values, simplify obfuscation, …

In 2014, the [RSoC]( http://rada.re/rsoc/ ) helped the project a lot, increasing the user base, defining specific tasks, teaching developers how to contribute to r2 and getting new features in a very nice way. This year we will try to get into the [GSoC]( https://developers.google.com/open-source/soc/?csw=1 ).

Pancake fixed some glitches in the RConsCanvas api to make the ASCII graphs better, and there are plans for future reusable APIs to construct interactive ascii-art graphs from the shell. So expect a blogpost about them in a near future.

[Pwntester]( https://twitter.com/pwntester ) is working *hard* on an awesome [web interface]( http://radare.today/the-new-web-interface/ ). Which is available at the [r2cloud]( http://cloud.rada.re )

Thanks to pancake's [sdb]( https://github.com/radare/sdb ), the analysis is now way more efficient, thanks to *O(1)* access times. The analysis is not yet fully ported to SDB, but it's in a transitional state which allows to interactively manage the database contents, contain memleaks and simplify in memory and disk data structures.

There's already support for debugging x86-32/64 Windows Kernels via the GDB and WinDBG support. But we have plans to greatly enhace it, extending the support for more platforms. In fact, It is now possible to also debug iOS applications from commandline using the XCode simulator (x86 only) with r2.

If crowell and jvoisin manage to keep each other uptight, you can also expect some enhancements in the exploitation area!

Since our [great lead dev]( https://twitter.com/trufae ) will start working for a mobile security company, you can expect massive improvements for analyzing and debugging applications for mobile operating systems.

In fact r2 is the first reverse engineering tool out there supporting Swift demangling of symbols (wip), and there have been a recent update for iOS and Android platforms (checkout playstore and cydia).

There's also an ongoing work on removing GPL'd code out of the r2 codebase (and moving it into the new [radare2-extras](https://github.com/radare/radare2-extras) repository).

We have plans to continue testing and using [Capstone](http://capstone-engine.org/) as a disassembler, we are using it by default for all the supported platforms, but there are some architectures (like MIPS16) which are not yet supported and we prefer to keep both plugins (GNU and Capstone) ones, to be able to compare them easily to find new bugs, compare speed, size and memory consumptions.

So it's up to the user to make his experiments. It's quite clear that Capstone is fixing bugs very fast and it has been proved to be quite reliable for most uses (so, we will try to be as proactive as possible to delegate the core disassembler maintainance to others).

And as usual, thanks to our inside fuzzer, and to our testsuite and to coverity, better code, less warnings, less memleaks, less bugs!

If you want to be part of this, we've got [some easy bugs]( https://github.com/radare/radare2/labels/easy ) to fix.