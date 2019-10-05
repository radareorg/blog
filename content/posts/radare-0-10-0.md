+++
date = "2016-01-21T23:30:07+01:00"
draft = false
title = "Radare 0.10.0"
slug = "radare-0-10-0"
aliases = [
	"radare-0-10-0"
]
+++

On Monday 16th, we released a new version of radare2, the *0.10.0*, codename *NOLAN*. Since you might be a but too lazy to read [every single commit](https://github.com/radare/radare2/commits/master), we’re going to highlight some cool new features together!

# Numbers
Thanks to more than 100 contributors who issued more than 2000 commits, here is what changed:

```
$ git checkout 0.10.0 && git diff 0.9.9 --shortstat
 1095 files changed, 80695 insertions(+), 40792 deletions(-)
```

We would like to thanks all contributors, especially the newcomers, that made this release possible. You can find a (hopefully) complete list of them in the [AUTHORS]( https://github.com/radare/radare2/blob/master/AUTHORS.md ) file.

# GUI
Every time someone says the word "radare2", someone asks about a GUI. Every
single time. There are already a lot of different interfaces, some are
web-based, some are in Qt, Gtk, tk, … some are public, some are privates, …
but not a single one is as powerful as the command line.
So for now, the recommended way to use radare2 is, the command line.

Another GUI-tradition is that our great leader pancake writes a new one for
each release. This time, it's implemented on top of the NodeJS r2pipe API and
it's called [blessr2](http://radare.org/r/blessr2.html) in honor of the `bless`
nodejs api to create beautiful and portable console user interfaces.

![blessr2](/blog/images/blessr2.png)

Since it took us a very long time to do this release, he had the time to
implement a second one, using Material design,
which turns out to be the default one for the Android and FirefoxOS applications.

# Stability
We spent a lot of time fuzzing radare2, collecting binaries and writing tests to improve radare2’s reliability. We even harvested similar projects bugtracker to see how well radare2 would deal with binary that broke them. Currently, we have something like 2000 tests dedicated to commands, and most of disassemblers have a 100% coverage.

About the testsuite, you may notice that it’s much more quick to run it now. We managed, on [travis-ci]( https://travis-ci.org/radare/radare2 ), to go from 1h30 for only gcc on linux, to 20 minutes for clang on OSX, and gcc+clang on linux. No more excuses for not running the testsuite before a commit.

You might  also be happy to know that radare2 now successfully compiles on [tcc]( http://repo.or.cz/tinycc.git ), the *tiny C compiler*. This might be useful if you’re compiling radare2 on weird platforms. Please be sure to use tcc from git too :) Moreover, radare2 tries as hard as it can to run on your-super-weird-platform-that-no-ones-has-ever-head-off, we implemented the `cp` and `mv` commands, since you might not find those everywhere.

Thanks to revskills for spending time fuzzing and reporting several parts of r2.

# Better support for iOS

Radare2 comes with some new features that will make iOS reverse engineers happy:

* `asm.emu` will tell you which objc_msgSend apis and syscalls are called
* Better emulation of Thumb, aarch64 and arm32
* Supports r2pipe in Swift, known to work on tvOS, watchOS, iPhone and OSX.
* Native OBJC parser implementation, no need to use `class-dump` tool anymore!
* Some enhancements in process memory dumping
* Supports tfp0 to read/write kernel memory if kernel is patched properly
* Exploit an iOS<=8 vulnerability to read 
* Code Signing is now done properly, updated instructions.
* Add support for nativelly running on Apple Watch (without jailbreak).
* Some random debugger bug fixes, still not fully working on iOS
* List memory modules, not just memory maps
* Unaligned instructions are different than the invalid ones
* MACH0 Crypto information is now accessible via SDB

ElCapitan users will get a bit pissed of because they are no longer able to
debug `/bin/ls`, because Apple's SIP will block debugging binaries found in
system directories. The solution for this is to copy them into your home :P
Also, default installation path has changed from `/usr` to `/usr/local`.

# Debugger

This release was supposed to focus on the debugger, fixing many issues, and
adding some new <s>bugs</s> features, like:

* Support for memory-access hardware breakpoints
* Much better Windows 32 and 64bit debugger support
* List opened handles and Windows using `dd`
* Rarun2 supports pipe execves in std file descriptors
* Remote debugging via IO plugins work a bit better now
* 3 different backtrace algorithms, configurable at runtime

![dbg](/blog/images/webui.png)

# Memory usage
It seems that no one ever took care of radare2 memory consumption before, because it was still lower than its competitors/alternatives. But for this release, radare2 went on a diet : it now consumes 3 to 5 times less memory !

# Pretty graphs
Our beloved *ret2libc* spent a lot of time rewriting graphs engine from scratch, with overlaps handling and better colours ! See how cool this is:

![graph](/blog/images/graph.png)

# New architectures support
We know a lot of people are using radare2 because it supports a lot of funky/exotic/awful/funny/scary architectures.

Remember when we added support for the famous [6502 cpu]( https://en.wikipedia.org/wiki/MOS_Technology_6502 ) in the last release? This time, we added analysis support and opcode description (with `?d`), because not everyone is fluent in 6502 assembly code. And even more, since we know some of you just care about the meaning of the code and not the beauty of the assembly listing, we added pseudo-decompiler support. Yes, we have a pseudo-decompiler for 6502.

Did you know that we have a contributor named [condret]( https://github.com/condret ) that really likes the *pokemon* game on gameboy? This is why he’s pushing ESIL, implemented a fancy gameboy disassembler, and for this latest release, he wrote a gameboy assembler! You can now craft your own shellcodes, or, if you’re crazy, games, for gameboy, with radare2.

We also improved AVR support, with analysis (radare2 analysis is generic, so it’s pretty easy to add its support for an architecture), an assembler, ESIL so you can emulate it easily, and description. This led two people (namely [Alexander Bolshev]( http://2015.zeronights.org/speakers#bolshev) and [Boris Ryutin]( http://2015.zeronights.org/speakers#ryutin )) to do worksops at [ZeroNights]( http://2015.zeronights.org/workshops.html ), [t2.fi]( http://t2.fi/schedule/2015/#speech12 ) and [S4x16]( https://www.digitalbond.com/s4/ ) conferences, about reversing and exploiting this architecture with radare2!

Also, we added support for assembling ARM and ARM64, ADN decoding (yes. It’s the `BCL` plugin in `r2pm`. You don’t know about `r2pm`? Keep reading then.), demangling for Rust binaries, Wii/Gamecube binaries, disassemblers for [LM32]( http://www.latticesemi.com/en/Products/DesignSoftwareAndIP/IntellectualProperty/IPCore/IPCores02/LatticeMico32.aspx ), [MCS96]( https://en.wikipedia.org/wiki/Intel_MCS-96 ), analysis and ESIL for PPC, V810 and RISC-V, ...

And since we have at least one Windows user, we also added support for Windows minidump format, aka `mdmp`, and windows-on-raspberry2-fileformat-it’s-almost-a-PE because apparently, it’s a real thing.

# Game Consoles
We have been also working in adding support for more game console ROMs:

* NES (nintendo-entertainment-system)
* SMD (sega megadrive)
* SMS (mastersystem/gamegear)
* DOL (wii/gamecube)
* GB  (initial support for assembling instructions)

Other new binary formats are now supported too:

* CGC executables
* MBN/SBL Android trustboot images
* Support for RPI2 PE Windows executables
* Windows Minidump (mdmp) files

# Bindings
Remember the bindings, and how much languages we supported? Remember when you had to read radare2’s source code to write a simple one-liner, and ended parsing a call to `system` with radare2, pipe, sed, pipe, tr, pipe, awk, pipe, sed ?
Yeah, us too. This is why we ditched (don’t worry, they are still there, but deprecated) the bindings, and created `r2pipe`. Since you like so much calling radare2 in `system`, this is exactly what is does: popping radare2, and piping commands to it.

This brings several advantages:

We don’t have to maintain a shitload of convoluted bindings (have you ever tried to use `ffi`?), we only have to implement a few commands per languages
You don’t have to read radare2’s source code if you don’t want to: if you know how to use radare2, you know how to use r2pipe! Append `j` to almost every command to get native JSON output! No need to use `sed`/`awk`/`tr`/… anymore!

For example:

```python
import sys
import r2pipe

r2 = r2pipe.open(sys.argv[1])
print('The five first instructions:\n%s\n' % r2.cmd('pi 5'))
print('And now in JSON:\n%s\n' % r2.cmdj('pij 5'))
print('architecture: %s' % r2.cmdj('ij')['bin']['machine'])
```

All r2pipe APIs has been updated to work on Windows, Linux and OSX. In addition, the new `native://` URI allows to use r2pipe api using the native C API instead of using pipes or sockets. This allows to reuse the same code but speeding up things a lot.

# r2pm
Radare2 had an implementation of 2048, a port-scanner, and even a secret ascii-penis, but now, it also has a package manager!

No, this is not overkill, stop complaining and keep on reading. Radare2 supports a lot of <s>useless</s> things. This is why we put non-code things into separate packages, that can be browsed/searched/installed/removed/updated with the new tool called `r2pm`.

```
$ r2pm
Usage: r2pm [cmd] [...]
Commands:
 -i,info                 r2pm -i # pkgs info
 -i,install <pkgname>    r2pm -i baleful
 -u,uninstall <pkgname>  r2pm -u baleful
 -l,list                 list installed pkgs
 -t,test FX,XX,BR BID    check in travis regressions
 -s,search [<keyword>]   search in database
 -v,version              show version
 -h,help                 show this message
 -c,clean                clear source cache
Environment:
SUDO=sudo                use this tool as sudo
R2PM_PREFIX=/usr         prefix for syspkgs
R2PM_PLUGDIR=~/.config/radare2/plugins   # default value, home-install for plugins
R2PM_PLUGDIR=/usr/lib/radare2/last/      # for system-wide plugin installs
$
```

Note that `r2pm -s` will show you every available package.

# License

We managed to remove the last bits of GPL licensed code in radare2! We’re not a complete LGPL project (some modules installable with `r2pm` have a different licenses, please pay attention to that). This means that you can use radare2 into your proprietary product, <s>while betraying</s> without giving the source to your users, but if you modify radare2, you need to publish the modifications. It might be easier for you to try to upstream them by the way ;)

# Usability
As usual, we’re going to pretend that every command has now a fancy colored help display, that we checked everything, yada yada yada. The truth is that now, <s>all</s> almost every commands is documented, and the help is coloured.

Also, there is even more autocompletion for commands like `pf` or `t`, that were super-tricky to use without.

You might remember hearing a loud noise a couple of months ago. This was when jvoisin was told that to have something like the `follow-fork-mode` in GDB, he had to find the syscall number for his architecture, break on this breakpoint with the `dcs` command, then attach to the right process (while having fun figuring which one it is), then continue. Yeah, it was a pretty loud scream. Fortunately, there is now the `dbg.forks` option for this :)

# ESIL
ESIL is our Intermediary Language, used for emulation, analysis, transformations, trolling, … This is why we added several new commands under `ae` (*A*nalyse with *E*sil), like `aeip` to set the ESIL eip to the current `eip`, ‘aef’ to emulate an entire function, added ESIL support for new architectures, held workshops, …
We even documented ESIL itself within radare2, since you can disassemble to ESIL with `e asm.esil = true`:

```
[0x00000000]> ae??
|Examples: ESIL examples and documentation
| +=     A+=B => B,A,+=
| +      A=A+B => B,A,+,A,=
| *=     A*=B => B,A,*=
| /=     A/=B => B,A,/=
| &=     and ax, bx => bx,ax,&=
| |      or r0, r1, r2 => r2,r1,|,r0,=
| ^=     xor ax, bx => bx,ax,^=
| >>=    shr ax, bx => bx,ax,>>=  # shift right
| <<=    shr ax, bx => bx,ax,<<=  # shift left
| []     mov eax,[eax] => eax,[],eax,=
| =[]    mov [eax+3], 1 => 1,3,eax,+,=[]
| =[1]   mov byte[eax],1 => 1,eax,=[1]
| =[8]   mov [rax],1 => 1,rax,=[8]
| $      int 0x80 => 0x80,$
| $$     simulate a hardware trap
| ==     pops twice, compare and update esil flags
| <      compare for smaller
| <=     compare for smaller or equal
| >      compare for bigger
| >=     compare bigger for or equal
| ?{     if popped value != 0 run the block until }
| POP    drops last element in the esil stack
| TODO   the instruction is not yet esilized
| STACK  show contents of stack
| CLEAR  clears the esil stack
| BREAK  terminates the string parsing
| GOTO   jump to the Nth word popped from the stack
[0x00000000]> 
```

Ho, I almost forgot to mention the new `asm.emuwrite`, `asm.emustr`, and `asm.emu` options! If you set them to true, radare2 will do its very best to improves the analysis with ESIL, but be careful, setting those variables may give you an über-verbose output.

# Extras

The radare2-extras repository has been growing because some stuff that was previously living in the main repository moved there. It comes now with a configure script and all the makefiles needed to build everything and make r2pm happy.

Some of the most interesting additions are:

## Unicorn
A lot of people are talking about [unicorn](http://www.unicorn-engine.org/), a
CPU emulator. While we think that ESIL is way better for everything and that
you totally should use it and contribute to radare2, we added support for it in
radare2, it’s as simple as `r2 -D unicorn /bin/ls`. In fact, since our great
leader pancake is great, he added support for it before its release, something like 6 months ago.

## Baleful

Skuater and thePope reverse engineered a crackme that was done on top of a custom virtual machine, so Skuater wrote the asm/anal/emulation plugins for it.

## BPF

Linux kernel packet filtering is done by a custom virtual machine that emulates code. r2 is now able to assemble, disassemble, analyze, emulate this new architecture. Thanks mrmacete!

## New bots

There are now new NodeJS bots for IRC and Telegram, ready to use in the `radare2-bindings/r2pipe/nodejs/examples/*`.

* r2tgirc : telegram-to-irc bot that communicates the #radare freenode channel with the Telegram's radare one.

* r2tg-bot : Radare2 bot for Telegram and connected to the cloud.

* r2irc-bot : IRC bot of r2 to use any binary in your system from the chat.


# Conclusion

Yet another successful release of radare2. If you want to join the party, please take a look at our [easy issues](https://github.com/radare/radare2/labels/easy ), we'll be happy to help you to help us, on [irc]( irc://irc.freenode.net/radare ), or on Telegram.

But more importantly, please, be sure you’ve pulled and rebuilt radare2 from git again, after finishing reading this post. Why? Because we can bet - someone already committed something new in the radare2 repository. Make it a new habit - periodically checking out the newest radare2 git. We can make reverse engineering great again!

# Thanks to

Hopefully i'm not missing anyone in the list of contributors. Thank you guys!

8tab, Adrian Pistol, Alvaro Muñoz, Avi Weinstock, Ben Gardiner, bhootravi, BlueC0re, bspar, bydo, Claude Hemberger, Darkkey, David Kreuter, David Manouchehri, Dāvis, deffi420, dequis, Derek Wood, dso, dukeBarman, earada, Felix Held, four0four, funktioniert, gk, Grigory Rechistov, iessa s alkuwari, inisider, isra17, Jeffrey Crowell, Jonathan Neuschäfer, Judge Dredd (key 6E23685A), Juha Kivekas, Kamil Rytarowski, Kevin Grandemange, Kiwhacks, mrdanielps, mrpink17, NighterMan, Ole André Vadla Ravnås, Oleksii Kuchma, Pierre Pronchery, PSi, qnix, Ricardo Quesada, Riccardo Schirone, Sam, Sascha Schirra, Serguey Parkhomovsky, seu, sghctoma, shengdi, shuall, Skia, Sushant, Sven Steinbauer, Sylvain Pelissier, szt, Taryn Hill, user, Vladislav Shtepin, Wilhelm Matilainen, Wladimir J. van der Laan, xambroz, yetmorecode, Y. Sapir, zlowram, ...

--pancake
