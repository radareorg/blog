+++
date = "2015-06-06T11:46:57+02:00"
draft = false
title = "Radare 0.9.9"
slug = "radare-0-9-9"
aliases = [
	"radare-0-9-9"
]
+++
Today, we're releasing a new version of radare2, the **0.9.9**, codename *Almost There*. Since you might be a bit too lazy to read [every single commit]( https://github.com/radare/radare2/commits/master ), we're going to highlight some cool new features!

# Numbers
Thanks to more than **50** contributors who issued something like 1700 commits, here is what changed:
```
$ git checkout 0.9.9 && git diff 0.9.8 --shortstat  
 839 files changed, 156490 insertions(+), 18885 deletions(-)
```

{pancake} I would like to give a special thanks to all the new contributors that made this release possible. You can find a complete list of them in the AUTHORS file. I am still the main developer, architect and maintainer of the project, but thanks to the increased popularity of the project i'm starting to delegate some tasks and handle the development from a better perspective, teaching newcomers, priorizing features, enforcing the testsuite and much more.

# Console
As you know, the current recommended way to use radare2 is its CLI interface. This is why we're doing our best to polish it. Our *Windows* user will be delighted to know that radare2 now [works great]( https://www.youtube.com/watch?v=pnZRsmY6xsw ) on their platform, and has almost reached feature-parity with *real* operating systems, with ground-breaking features like arrow-keys support or ^C to issue `SIGTERM`.

Most of the w32 enhacements were done by [Skuater](https://twitter.com/sanguinawer), who fixed and tested some bugs in the windbg plugin, implemented support for FPU and MMX registers on Windows and Linux-x86-32/64, enhaced the console input to work almost as well as in linux.

One of our core-contributors happens to be a [truecolor fanatic]( https://gist.github.com/XVilka/8346728 ), so now radare2 can support all 16,777,216 colors!

Among various console improvements, you'll find the new variables  `scr.wheelspeed` and `scr.responsive` to improve navigation.

We know that the learning curve for radare2 is *super*-steep, and we're sorry about this. The good news is that we checked that documentation was available for every single command, and wrote it where it was missing! You can as usual append `?` to your commands to get documentation about them.


# New architectures
## i4004
Let's go back in time to 1971. At this time, Intel released the first general purpose programmable microprocessor on the market, the [i4004]( https://en.wikipedia.org/wiki/Intel_4004)

![i4004 CPU](/blog/images/C4004_-Intel-.jpg)

It was a blazing-fast CPU, 740 kHz, able to directly address 640 bytes of RAM! So now, 34 years later, radare2 supports this CPU.
 
# LH5801
While we're back in time, did you know that we're supporting the good old [LH5801]( http://www.old-computers.com/museum/computer.asp?st=1&c=965 )? 

![pocket computer](/blog/images/shapre.JPG)

It's a 8bit CPU that was used in the first [pocket computer]( https://en.wikipedia.org/wiki/Pocket_computer )!
 
## z80
The previous [z80]( https://en.wikipedia.org/wiki/Zilog_Z80 ) disassembler was under GPL, had comments in German (Like [LibreOffice]( https://bugs.documentfoundation.org/show_bug.cgi?id=39468 ) and [systemd]( https://plus.google.com/+LennartPoetteringTheOneAndOnly/posts/VUzeRLf5g5m )!), was huge and a pain to maintain. Thanks to condret, we now have a clean, correct, LGPL-licenced z80 disassembler which is 75% smaller!

## Pebble
If you have a [pebble watch]( https://getpebble.com/ ), you can [now]( https://github.com/radare/radare2/commit/e35d55e18ab2bfe1d38b4a9398222531982f070a ) disassemble applications with radare2!

# Analysis improvement
Added two alternative analysis loops with several levers to tune some options like skipping nops at the begining of the function, detect functions by following calls, handle local variables, ELF PLT and Thumb detection are now supported for ARM and ARM64;
local-flags/function-labels are also back for every supported architecture.

PE relocations are now displayed in a *sexy* way:
![PE RELOC](/blog/images/reloc.png)
![PE RELOC 2](/blog/images/reloc2.png)

There is (basic) support for [CRIS]( https://en.wikipedia.org/wiki/ETRAX_CRIS ) analysis
![Cris anal](/blog/images/cris.png)

Also, can you [spot]( https://twitter.com/radareorg/status/580289536892252160 ) Dalvik-related enhancements?

# Commands changes
We changed some commands (for good), but since they were cryptic, you probably never used it before, so you won't even notice the changes. If you do, we would appreciate your feedback.

We also added a new ones, mostly subcommands of `p`. Can you find them? ;)

# Fixing bugs
Thanks to the amazing work of [maijin]( http://maijin.fr/ ), we now have our (ever-growing) testsuite running on [travis]( https://travis-ci.org/radare/radare2 ) to avoid regressions!

Also, [jvoisin]( https://github.com/jvoisin ) had fun fixing 75% of our coverity issues, bringing the current total to less than 150!

We also fixed bugs found by [shellcheck]( http://www.shellcheck.net/ ), [cppcheck]( http://cppcheck.sourceforge.net/ ), [valgrind]( http://valgrind.org/ ), and more!

# ESIL
Remember [ESIL]( https://github.com/radare/radare2/wiki/esil )? Our [IL]( https://en.wikipedia.org/wiki/Intermediate_language ). Come on.

Anyway, [condret]( http://runas-racer.com/main.html ) has been working hard on it, mainly working in the specs and gamebody support. [Nighterman]( https://twitter.com/nighterman ) has added features for x86 emulation, [pancake]( https://twitter.com/trufae ) for arm and mips and [dkreuter]( https://twitter.com/dkreuter_ ) for i8051... Emulation, here we come!

Also, congrats to [sushant94]( https://sushant94.github.io/ ) for his implementation of an ESIL to [REIL]( http://blog.zynamics.com/2010/03/07/the-reil-language-part-i/ ) translator and [dkreuter](https://github.com/dkreuter) for his [ESIL implementation for 8051](https://github.com/radare/radare2/blob/master/libr/anal/p/anal_8051.c)! Not bad for a first contribution, heh?

# Search 
We [already wrote]( http://radare.today/ropnroll/ ) about it, but [crowell]( https://github.com/crowell ) added regular-expressions support to the rop-gadget finder. Also please note that the separator is now `;`, and that you must quote *the whole command* when you use it.

Some people are using radare2 instead of [binwalk]( http://binwalk.org/ ) to run libmagic on unknown files. This is why we optimized a bit the speed and efficiency of the `/m` command.

# ASCII graphs
Remember when we [bragged about]( http://radare.today/initial-ascii-art-graph-layout/ ) the awesome ASCII-graph support in radare2? Well, today, thanks to [r0nk]( https://github.com/r0nk ), we'll brag again:

Graphs now have awesome colours by default:
![Graph colours](/blog/images/r2graphloop.png)

Of course, colours are supported in the minigraph too!

![Coloured minigraph](/blog/images/meh.png)

We've got two display mode for graphs edges. You can switch between them with the `e` key.

![Graph edge styles](/blog/images/r2graphmain.png)

# Teaching
Radare2 is not only used to reverse exotic binaries, or craft ingenious exploits : it's also used to teach computer science!

Radare2 comes with almost 250 fortunes, and while we think that they are super-fun, some might actually be offensive, or ill-suited for *formal* presentations. This is why we split them: you can now set `cfg.fortunetype` to `tips`, `fun`, `nsfw`, or any combination of them. We hope that this will help you to  avoid awkward situations while doing a presentation ;)

Since not everyone is fluent with weird instructions set, radare2 comes with an `asm.pseudo` option, to show instructions in a more obvious way.

You can also try our *proto-alpha-preprod* decompiler with `pdc`:

![decompiler](/blog/images/decomp.png)

# Debugger
## WinDBG
[TheLemonMan]( https://github.com/LemonBoy ) added support for [WinDBG]( https://msdn.microsoft.com/en-us/windows/hardware/hh852365 ), the ring-0 debugger of Windows. This means that you can not only debug drivers with radare2, but also virtual machines. Imagine, breaking, modifying and stepping Windows, with radare2!

![WinDBG support](/blog/images/windbg.jpg)

## Tracing
Thanks to [earada]( https://twitter.com/earada ), tracing is now working much better and can be displayed in the ascii-art and web graphs.

![tracing](/blog/images/tracing.png)

# Web interface
We already said a lot about our [new web interface]( http://radare.today/the-new-web-interface/ ), by [pwntester]( https://github.com/pwntester ), but I'm quite sure that you can't have enough of it:

- a miminap
- massive speedup
- interactive graphs
- even more contextual menu
- hexdump
- projects support
- type-edition
- variables renaming
- debugger support
- tracing

![tracegraph](/blog/images/tracegraph.png)

# r2pipe
Since radare2 is a fast-moving target, instead of using *traditional-bindings*, the recommended way to call radare2 from a foreign language is to use *r2pipe*, which is roughly an API to communicate with an instance of radare2 using HTTP, PIPEs, TCP sockets or STDIO to run r2 commands and get the output in a string.

We'd like to take this opportunity to remind you that you just have to add `j` to every single command to get the output in *JSON*. If you're parsing raw radare2 input by yourself, you're going to have a bad time.

Currently, we have stable and mature support for  Python (2+3), Go and NodeJS; but also support D, C#, Java, Ruby, Perl, Vala, NewList, Shellscript, Rust...

Packages for r2pipe are available from the python [pip]( https://pypi.python.org/pypi/r2pipe/0.6.5 ), ruby [gem]( https://rubygems.org/gems/r2pipe ), and node.js [npm]( https://www.npmjs.com/package/r2pipe ) package managers.

[r2pipe]( https://github.com/radare/radare2-bindings/tree/master/r2pipe ) offers a simple interface for running r2 commands over a pipe, tcp or http connection and get the output in a plain string or a JSON object. Also it have been integrated with rlang, so you can run those scripts from the shell like if it was a plain r2 script:

	r2 -i stuff.py /bin/ls
    [0x8040580]> . stuff.py



# Misc
Most of the radare developers are using [vim]( http://emacs.rehab ), but some of us prefer [emacs]( https://www.gnu.org/software/emacs/ ). This is why there are now vim *and* emacs keybindings in visual mode!

Thanks to [Aaron Puchert]( https://github.com/aaronpuchert ), we've got a new assembler for x86, cleaner, and more efficient! But we are still using the x86.nz assembler plugin which supports more instructions, and bear in mind that you can also use the x86.olly or x86.as.

# Ok, what's next?
It depends of what you're going to implement of course! Now that we have a fantastic testsuite, it's way easier to contribute. The [next release]( https://github.com/radare/radare2/milestones/1.0.0 ) should focus on improving the debugging capabilities, but since we're an open-source project, it depends what our contributors are up to.

# Misc
## Build
Thanks to [pancake]( https://twitter.com/radareorg ), build time have been reduced (especially on Windows, where you can expect a 30% reduction!). This is why it only takes [3m37s](https://twitter.com/radareorg/status/588124975976095745 ) to build radare2 from git on the [UbuntuPhone]( http://www.ubuntu.com/phone/devices ).

The [android application](https://play.google.com/store/apps/details?id=org.radare2.installer&hl=en) has been updated, you can now build radare2 :

- on iOS 8.3 and its simulator
- on latest OSX statically
- without GPL plugins

## TV
Did you know that radare2 [was shown]( https://twitter.com/radareorg/status/582610366561140736 ) on a national (spanish) TV chain thanks to [Gabriel Gonzalez]( https://twitter.com/GabrielGonzalez) from [IOActive]( https://twitter.com/IOActive ).

By the way, if you're using radare2 at work, we'll be delighted if you [let us know]( http://radare.today/who-uses-r2/ ) about ;)

## Bonus screenshots

![Edge style comparison](/blog/images/r2graphlines.png)
Comparison of the two edge styles.

![R2 3D logo](/blog/images/r2-3D-logo.png)
Awesome stereograms!

![Mazda](/blog/images/mazda.jpeg)
Of course radare2 runs on your Mazda!
Yes, it's a car :)
