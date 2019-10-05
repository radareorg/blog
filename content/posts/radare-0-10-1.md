+++
date = "2016-02-29T23:30:07+01:00"
draft = false
title = "Radare 0.10.1"
slug = "radare-0-10-1"
aliases = [
	"radare-0-10-1"
]
+++

# radare2 0.10.1 - Release Notes

Six weeks ago, when our great leader pancake announced "a release every 6 weeks", everyone was a bit, well, [surprised]( https://twitter.com/r2gif/status/693462019610132480 ), but it seems that we did it.

But first, some numbers:

* Codename: `solid chair society`
* Weeks: 6
* Commits: ~280
* Issues Fixed: 50
* Contributors: 38
* New contributors: 10
* New easter-eggs: 1

This 0.10.1 release pushes other updates for:

* [sdb](http://radare.org/get/sdb-0.10.1.tar.gz)
* [acr](http://radare.org/get/acr-0.10.1.tar.gz)
* [radare2](http://radare.org/get/radare2-0.10.1.tar.xz)
* [radare2-bindings](http://radare.org/get/radare2-bindings-0.10.1.tar.xz)
* [radare2-extras](http://radare.org/get/radare2-extras-0.10.1.tar.xz)

Also binary builds for [Windows](http://radare.org/get/pkg/radare2-w32-0.10.1.zip) and [OSX](http://radare.org/get/pkg/radare2-0.10.1.pkg) are also available.

This is great, since it means that our downstream people who puts radare2 into package manager will be able to push updates quicker (yes, I'm looking at you, [debian](https://packages.debian.org/jessie/radare2)).

This also means more release party, which is a good thing. There wasn't an special focus on anything during that last 6 weeks, but if I had to comment on this release, I would say that its theme would be "compiling on windows", and "usability". Or something like that.

Anyway, here is the human-readable changelog:

- Variables and flags can now be renamed in cursor mode [asciinema](https://asciinema.org/a/dp49kdvy1y1lykbb4n6fsxurb)
- Optimized GDB connectivity, now its 10x faster!
- print signed base 10 hexdumps with pxd[1,2,4]
- radiff2 -C to compare checksums
- Lot of work towards the mach-ification of the OSX/iOS debugger by alvarofe
- more polished cursor movements in Visual mode
- Better ARM and Thumb code analysis and emulation
- Added disassembler support for Microblaze architecture
- Updated unicorn plugin to be in sync with git
- Various enhancements in the Visual mode 
- backward disassembly uses RAnal info for better offset computations
- `asm.bbline` uses RAnal info to have precise results
- fix bug in `env.sh` when using more than 9 arguments
- Mingw compilation improvements
- preliminary support of [XNU]( https://en.wikipedia.org/wiki/XNU ) debugging
- ESIL support for [v810]( http://www.archaicpixels.com/blog/images/4/40/U10082EJ1V0UM00.pdf )
- radare2 does now compile in [appveyor]( https://ci.appveyor.com/project/radare/radare2-shvdd ): no more excuses for broken commits on windows!
- [Lanai]( http://lists.llvm.org/pipermail/llvm-dev/2016-February/095118.html ) (the secret CPU used by Google) support
- a new shiny [xtensa]( http://ip.cadence.com/ipportfolio/tensilica-ip/xtensa-customizable ) CPU analysis backend
- change local variables/arguments format names (`ebp-0x10`, `ebp+0x13` becomes, respectively, `local_10h` and `arg_13h`) and now it works too when asm.ucase is set.
- add `Vdn` option to rename a flag/function/local variable/local argument used in the current instruction
- refactoring of `RFlag` + better names for functions when there are symbols
- `ahi` now supports IPv4 and syscall
- various optimizations and bugfixes
- opcodes descriptions for v810, propeller, riscv, tms320, lm32, i4004, i8080, java, Malbolge, SH-4, M68K, ARC and LH5801 (that you can access with `?d` or e `asm.describe=true`)
- `axg` to get a graph of the function xrefs to reach a specific point.

![](http://i.imgur.com/fbExjw0.png)

![](http://i.imgur.com/eE29fxK.png)

![](http://i.imgur.com/v7zs6Oz.png)

![](https://asciinema.org/a/dp49kdvy1y1lykbb4n6fsxurb)

![Lanai CPU](https://regmedia.co.uk/2016/02/09/myricom_lanai.jpg)

Known regressions and future work
-------------------------------
The webui graph stopped working on Google Chrome because they have deprecated a js function to manipulate SVG which was used by the joint.js library, the webuis will be distributed in a [separate repository](https://github.com/radare/radare2-webui) and dependencies will be maintained using bower/grunt/npm. This way we will solve the license problems some distros (Debian) complained for not packaging the webuis because of non-free and confusing uglified js blobs. This will hopefully attract more web developers.

Debian, Docker, Void, FreeBSD, Sabotage and other distros raised the interest in our project, so, the 6week release cicle will hopefully fix the problem of having very old packaged versions of r2.

Windows binaries from appveyor still need to be fixed thus the windows installer.

There are some interesting wip patches to be included in the next [release 0.10.2](https://github.com/radare/radare2/milestones) scheduled for April 11th.

Also, it is important to note, that some people started to work on the GSoC microtasks even before knowing if we are accepted this year. This is a good sign which clearly shows the growing, healthy and brave community we have.

Special thanks to:
----------------
* alvaro felipe: for fixing some bugs and enhancing the XNU debugger
* xvilka: finally getting the windows builds happy again
* maijin: for reviewing issues and adding more opcode descriptions
* deffi420: to find and fix some tiny, but important bugs in SDB
* condret: for working on the SIOL branch that will hopefully be merged soon.
* crowell: enhacing the local variables experience
* ret2libc: fixes a bug in dietline, rewrote flags, metadata, better midflags and cursor movement.
* mballano: for commiting for the first time, making RAP:// more consistent.

Have fun with this new release and keep up hacking!

--pancake
