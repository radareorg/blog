+++
date = "2014-11-12T17:26:07+01:00"
draft = false
title = "Radare 0.9.8"
slug = "radare-0-9-8"
aliases = [
	"radare-0-9-8"
]
+++
Eight months ago, radare2 0.9.7 was released; today, we're happy to announce radare 0.9.8!

In details and numbers:

* More than 2500 commits
* More than 120 new users in #radare (+300%)
* About 60 [contributors]( https://github.com/radare/radare2/pulse ).
* 13 colorscheme themes.
* 8 months
* 1 [great leader]( https://twitter.com/trufae )
* One [homepage]( http://www.radare.org )
* A version nubmer: 0.9.8
* A **[soundtrack]( http://radare.org/get/Neuroflip-BabylonRocket-OriginalMixForR2.mp3 )**! Thanks to [neuroflip]( https://twitter.com/neuroflip )!

## Downloads:

* [Sources]( http://rada.re/get/radare2-0.9.8.tar.xz )
* [Bindings]( http://rada.re/get/radare2-bindings-0.9.8.tar.xz )
* [Valabind]( http://rada.re/get/valabind-0.9.0.tar.gz )
* [SDB]( http://rada.re/get/sdb-0.9.2.tar.gz )
* [Git repository]( https://github.com/radare/radare2 )

Since you surely can read the 2500 commits by yourself (or the [detailed changelog]( http://rada.re/get/changelog2-0.9.8 )), we're just going to highlight some cool new features and improvements:

## New platforms!

r2-0.9.8 now builds for [Android L]( https://en.wikipedia.org/wiki/Android_Lollipop ), which enables two new architectures:

* Android mips64
* Android aarch64

![android screenshot](/blog/images/adroid.jpg)

Also, we are now supporting crosscompilation for iOS devices. No need to use jailbroken devices as main builders. The `sys/ios-sdk.sh` script supports the very latest XCode from Yosemite.

We are also happy to get r2 running on [Haiku]( http://haiku-os.org/ ) and it's great that it's now finally possible to compile it natively on Windows7 using [mingw32]( http://mingw.org/ ). Also xvilka helped to catch and fix the remaining bits to get r2 compilable with [Cygwin]( http://cygwin.com/ ) too.

Javascript is also another supported target platform to compile r2 to, [SDB]( https://marketplace.firefox.com/app/sdb-1 ) and [Rax2]( https://marketplace.firefox.com/app/rax2 ) have been already pushed into the [FirefoxOS marketplace]( https://marketplace.firefox.com/search?author=pancake ).

## New architectures
We're now supporting:

- Java, thanks to [dso]( http://thecoverofnight.com/ ). Ok, it's not new, but it was *greatly* enhanced!
- cr16 and msp430, thanks to [montekki]( https://github.com/montekki )
- Nintendo DS, thank to [aortega]( http://blog.badtrace.com/ )
- tms320(c55x, c55x+, more is coming), thanks to [milabs]( http://brainstorage.me/milabs )
- m68k - fixed some segfaults from the imported disassembler and use it to reverse engineer [some Street Fighter roms]( http://pof.eslack.org/2014/04/08/hacking-super-street-fighter-ii-turbo-part-2/ ).
- Spc700 - (supernes sound-chip)
- Propeller
- Every arch supported by capstone that we didn't already had, like SystemZ

## Better Debugger

![debugger](/blog/images/regstacklisting.png)

Also, we have reworked on w32 to make the native debugger work again and support hardware breakpoint registers on x86. (thanks Skuater!)

The tracepoints have been implemented, Now each breakpoint can be be toggled to become active/disabled or be used as a tracepoint or breakpoint.

Lot of work have been done to make the GDB remote work and support X86, MIPS and ARM for qemu and gdb-server targets.

Bear in mind that GDB is a totally fucked up protocol with lot of incompatibilities (yes, more than ELF), and each new target depends on several sub-features to be implemented or handled differently in the client side. defragger did a great work testing and implementing them, but there are still more targets to be supported.

## SDB integration
[SDB]( https://github.com/radare/sdb ) is our key-value database system, that we're integrating into radare2. This simplifies the code, and increases the performances in many areas.

![SDB](/blog/images/sdb.png)

You can play around with SDB using the `k` commands.

## Capstone
We're now using [capstone]( http://www.capstone-engine.org/ ) to disassemble several architectures. This simplifies maintainance on our side, and bring you complete support for several architectures and their extensions (hello AVX, FPU, ...).

If you are having disasm or analysis problems, switch to the old udis86 or GNU plugins to compare and send us your feedback. We'll fix them as soon as possible.

## Quality
Thanks to [coverity]( https://www.coverity.com/ ), we fixed a lot of bugs. We're also using [ASAN]( https://code.google.com/p/address-sanitizer/ ), [valgrind]( http://valgrind.org/ ), [Jenkins]( http://ci.rada.re/ ) and a complete [testsuite]( https://github.com/radare/radare2-regressions ) to improves r2 efficiency and reliability.

We also cleaned up lot of warnings and memory corruption issues by using different fuzzers ([radamsa]( https://code.google.com/p/ouspg/wiki/Radamsa ), [melkor]( http://blog.ioactive.com/2014/11/elf-parsing-bugs-by-example-with-melkor.html ), [nightmare]( https://github.com/joxeankoret/nightmare ), [zzuf]( http://caca.zoy.org/wiki/zzuf ), …), to make the RBin plugins much more reliable on fucked up targets. We know there are still some corner cases to fix, and we will address them in the next bugfix release 0.9.9.

The R_IO layer have been heavily tested, reviewed and partly rewritten to be more stable (thanks condret!)


## DWARF
![](/blog/images/dwarf.png)
Montekki spent some time to add support for [DWARF]( http://dwarfstd.org/ )! Now you can debug your favourite ELF binary (almost) without reading assembly! 

## ESIL
[ESIL]( https://github.com/radare/radare2/wiki/ESIL ) stands for *Evaluable Strings Intermedate Language*. It aims to describe a Forth-like representation for every opcode; the goal within radare being emulation, in order to improve the analysis.
![](/blog/images/esil.png)

With ESIL it is now possible to:

* Debug Brainfuck applications
* Partially emulate Gameboy, X86, ARM and MIPS.
* Define software watchpoint conditions.
* Spawn complex search expressions
* Analyze code without messing with arch-specific and low level struct issues.

We will continue to enhance the ESIL support to  emulate code in a better fashion, and we plan to use it for injecting code on child processes and more.

# Exploitation
![pop pop ret](/blog/images/rop_pwn1.png)
- Analyze an specific memory address with `ai`
- Take references from registers and memory with `drr/pxr` (*à la* PEDA)
- ROP gadget searcher (`/R`)
- Mitigation detections (in `i`)
- [De Bruijn]( http://www.offensive-security.com/metasploit-unleashed/Writing_An_Exploit ) patterns
- `search.in` eval variable can be used to specify where to search for patterns (for example, inside heaps or executable pages).

## ASCII art graphs
Spawned by default when pressing `V` inside the *Visual mode* when placing the cursor on an already analized function. For callgraphs use `VVV`.

![graph](/blog/images/Screenshot-2014-04-29-01-51-31.png)

Graphviz, JSON and HTML5 interactive graphs are still supported, but not used by default.

In addition, some basic debugging can be done inside the ascii graph graphs.

The visual mode has been enhanced to better handle cursor, enable feedback mode (useful for screencasts), navigating references with `x` and `X` keys, and support the mouse wheel for scrolling (which can be disabled with the `scr.wheel` eval variable).

## RSoC
Since we've been rejected from the [GSoC]( https://developers.google.com/open-source/soc/ ), we've launched [our own]( http://rada.re/rsoc/ ) Summer of Code. Which turned to be kinda profitable from the point of view of new contributors, feedback, testing and getting new features ready for the masses. Congratulations to everyone who attended and participated!

We will soon release another post explaining all the details of the finalized RSoC.

### Structures support
[Skia]( http://libskia.so/ ) did great to implement structures support, a bit *à la* 010Editor.
![structure](/blog/images/687474703a2f2f7777772e6c6962736b69612e736f2f7075622f72322532306c696e6b65642532306c6973742e706e67.png)

Now with `r2 -nn` it is possible to analyze the file header structs using the `pf.`, `pxa` and other related commands.

```
$ r2 -nn /path/to/bin
```

### FLIRT and YARA support

[jfrankowski]( http://failhard.org/ ) did a great RSoC, and improved [yara]( https://plusvic.github.io/yara/ ) support in radare2, and also added the [FLIRT]( https://www.hex-rays.com/products/ida/tech/flirt/in_depth.shtml ) one! Currently, radare2 is only able to use existing signatures, but feel free to drop us a patch to build our own, using radare!

Yara3 support is almost there, but we prefer to release for a welltested yara2 version and push the upgrade in 0.9.9.

### PDB support
[inisider]( https://github.com/inisider ) implemented a standalone library to handle [PDB]( https://support.microsoft.com/kb/121366 ) files, and integrated it into radare2. You can now analyse/debug PE with much more ease.

    > .!rabin2 -rP test.pdb

## Integration and bindings

The IDA to r2 Python script has been also updated.

[Duktape]( http://duktape.org/index.html ) have been included to support inline Javascript scripting, which turns to enable portable programs that can run in NodeJS, the r2 shell or the web browser without modifications.

We have also integrated support for GZip files which can be now slurped or loaded without depending on external programs (as we already did with ZIP).

## Documentation and PR
- the [radare2 book]( https://maijin.github.io/radare2book/ )
- [this]( . ) blog ;)
- We were at [some con]( http://radare.today/we-were-at-hack-lu-2014/ ), and did [some workshops]( http://radare.today/trainings-and-translations/ )

# Forensics

libmagic support has been enhanced to the point that we can already replace some features of [binwalk]( http://binwalk.org/ ) to analyze router firmware images.

Also, several commands for searching with asteroids have been added, in order to make data analysis much better.

Also, commands like `rabin2 -zzz` allow to dump all strings from a file without storing the results in memory which works great for dumping harddisks or memory snapshots.

# R2Pipe and JSON
Since this release you now can use a very fast kind of bindings - r2pipe, which is actually piping output of the commands inside the chosen language wrappers instead of directly calling C function from the library. Also it is very easy to use, because if you know radare2 commands.

A lot of commands spit JSON when typing a `j` at the end of the command. This is very useful for exporting information from r2 or processing it from the webui.

In addition a bunch of new bindings have been created under the name of `r2pipe` which use r2 with `-0` flag to send commands and receive output asyncrunously.

https://www.npmjs.org/package/r2pipe

That solution tends to be far more realiable than depending on [SWIG]( http://swig.org/ ), faster than using FFI and easier to make it distributable or work remotely.

# Bonus
Since we're a bunch of nice people, we also implemented:

- An ascii version of [2048]( https://en.wikipedia.org/wiki/2048_%28video_game%29 )
- A lot of colourschemes
- Help is colourized
- Greatly enhanced analysis
- Nearly all commands can now output JSON

# Screenshots!

![colourscheme](/blog/images/B0IyY38CcAIzxdy.png)
A new colourscheme

![pxa](/blog/images/meh.jpg)
Coloured hexdump

![Gameby patching](/blog/images/gb.png)
Because patching Gameboy ROM with radare2 is fun!

![Demangling](/blog/images/demangling.jpg)
Prototype of demangling

![sonare](/blog/images/sonare.png)
First screenshot of [sonare]( https://github.com/sapir/sonare ), a Qt-based disassembly viewer based on radare2.