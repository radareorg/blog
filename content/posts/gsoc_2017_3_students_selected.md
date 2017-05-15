+++
date = "2017-05-15T02:45:16+02:00"
draft = false
title = "GSoC 2017 selection results"
slug = "gsoc_2017_selection"
aliases = [
 "gsoc_2017_sel"
]
+++

# Google Summer of Code 2017

![](https://i.imgur.com/u6c06A9.png)

Good morning ladies and gentlemen!
We're happy to present you our 3 students, who passed all the hurdles of [GSoC'17](https://summerofcode.withgoogle.com/) selection and successfully shown their passion for radare2 development. Two projects are related to debugging capabilities of radare2 - both local and remote, and the 3<sup>rd</sup> one will improve the most wanted platform support - Windows.

Here they are introducing themselves and their projects:

# Rkx1209

Hi. I'm [rkx1209](https://rkx1209.github.io/) from Japan and I have been working on *Timeless Debugging support* for radare2's debugger. Over the last couple of weeks I've implemented two new commands `dsb` (debug step back) and `dts` (debug trace session). Thanks to those contributions, radare2 is now supporting native timeless debugging. 
You can use `dts` to save the program state at any moment and step one back by with `dsb` until the previously saved program state point. You can see a demo of this preliminary work [here](https://asciinema.org/a/7i8rcl6eyu0xsuc249lzqhi4m). 
Over the remaining duration of GSoC I'm gonna do following tasks to improve current implementation.

1. Implementing new diff style format of program snapshot for efficence
2. Adding checkpoint system that automatically saves snapshots at some execution points
3. Adding 'dcb' (debug continue back) command.
4. Implementing watchpoint function that is more efficent when it's used with back step commands.

I'll also try to fix some issues of current implementation.(e.g. In Mac OS X, sometimes, `dts` crashes. )
If you want to follow my progress, you can see it on my [blog](https://rkx1209.github.io/).

![](https://i.imgur.com/NgGi5yj.png)


# Xarkes

Hi, I'm Antide from France, also known as [xarkes](https://twitter.com/xarkes_). I'm a 20 years old IT student, currently 1st year in [ENSIMAG](http://ensimag.grenoble-inp.fr) (Grenoble) Engineering School. 
For this Google Summer of Code, I'll be working on improving the support of radare2 under Windows. The first part of my GSoC will be dedicated to a proper building and testing for Windows environment using [AppVeyor](https://ci.appveyor.com/project/radare/radare2), to make development easier and more stable. During the second phase, I'll improve the support of PDB files, improve DLL files analysis, and debugging. Finally, I'll complete the WinDbg protocol support for radare2 and finish the current minidumps loader plugin.
With all of this, radare2 should be more attractive for Windows users, in fact I already started working on making radare2 compatible with Visual Studio compiler, so it's easier for any Windows developer to contribute to it.

![](https://i.imgur.com/xaHKpgp.png)
![](https://i.imgur.com/MYYzXPA.png)


# Srimanta Barua

Hi, I'm [Srimanta Barua](https://github.com/SrimantaBarua), a third year undergraduate student from India. I'm relatively new to reverse engineering and radare2 is the first real tool I've used (I can't afford to pay for an IDA license), so the love has stayed strong since. I've contributed small bits of code over the last few months, primarily fixing bugs in analysis. My contributions can be found [here](https://github.com/radare/radare2/pulls?q=is%3Apr+is%3Aclosed+author%3ASrimantaBarua)
and [here](https://github.com/radare/radare2-extras/pulls?q=is%3Apr+is%3Aclosed+author%3ASrimantaBarua).
 An example here with my contribution of adding display of strings in disassembly of PIC binaries:

![](https://i.imgur.com/bvYq0Sa.png)

For GSoC '17, I will be working on a [gdbserver](https://en.wikipedia.org/wiki/Gdbserver) implementation inside radare2, and simultaneously improving the gdb remote client. The core goals for the summer include support for basic debugging commands like break/continue/step, for x86/ARM/MIPS/PowerPC (at least), and being able to handle commands from r2/gdb/LLDB/IDA Pro.

The interface for starting and using gdbserver inside r2 will work similar to that for RAP or HTTP. This project involves first extending the io_gdb plugin to support accept, listen and other calls, and different options for open. Then, I will work on extending the code for the gdb remote protocol.

As an additional goal, I will be working on I/O caching to prevent multiple redundant reads. I'll also work on loading and rebasing symbols to honor the remote memory maps. I'll also take up other issues on the r2 repository.

It promises to be a good summer.

![](https://i.imgur.com/DHPNM3k.png)

