+++
date = "2016-04-11T23:30:07+01:00"
draft = false
title = "Radare 0.10.2"
slug = "radare-0-10-2"
aliases = [
	"radare-0-10-2"
]
+++

# radare2 0.10.2 - Release Notes

Codename: Panamake

As usual, some numbers first:

```
Contributors: 48
Commits: 480
Issues: 135

Grep stats:
* Fixes: 269
* Add: 107
* Enhance: 7
* New: 7
* Esil: 18
* Anal: 36
* Leak: 15
```

Contributor commit counter: (sys/pie.sh)
```
$ sys/pie.sh 0.10.1 | sort -un | tail -n 13
1	Adrien Garin
2	Adr1
3	Kitsu
4	Darredevil
5	Anders Kaare
6	Aneesh Dogra
7	Evan Shaw
8	Jeffrey Crowell
12	Maijin
16	Anton Kochkov
36	oddcoder
46	Álvaro Felipe Melchor
237	pancake
```

Special thanks from pancake to:

* [@revskills](https://twitter.com/revskills) for the massive fuzzing
* [Google](http://radare.org/gsoc) for the GSoC
* [Ghostbar](https://twitter.com/ghostbar) for being the new Debian maintainer and update r2 packages
* [Nibble](https://twitter.com/nibble_ds) for coming back
* [Alvaro Felipe Melchor]( https://alvarofe.github.io/ ) for the elf relocs, dyldcache and the massive bugfixing.
* oddcoder for being the most active student
* Daniel Dominguez for the initial coredump support

This release is much bigger than we ever thought. Thanks to Google and GSoC applications process we've got an honest amount of a students' contributions, who implemented a few file formats, improved analysis and fixed a bunch of issues. 

Highlights
----------

- Add `r2 -d` and `-R` shortcuts to simplify loading rarun2 profiles and using remote debugging plugins
- Support for cryptography (blowfish, rc2, rc4, aes, xor, ror, rol)
  - `woE`/`woD`
  - `rabin2 -E`
  - Adding `wo*y` commands using clipboard instead of [val]
- Better PE and ELF parsers
- New easter-egg!
- dyldcache extractor is working again
- Support for [BOCHS]( https://en.wikipedia.org/wiki/Bochs )
- [Coredump]( https://en.wikipedia.org/wiki/Core_dump ) support for iOS and OSX
- New fileformats, namely Python bytecode and Flash files
- Improved analysis and emulation thanks to ESIL on x86, ARM and MIPS
- New `make menu` to choose plugins to build
- Add `?E` clippy echo and use it in ????
- xrefs and types are now properly saved/restored from projects

New R2PM packages
-----------------
- ramoji2
- www-t and www-p
- syms2elf

Better Disassembly
------------------
- Add `asm.spacy` and `asm.flgoff`
- *noreturn* function database is much more reliable now
- Summary mode (`pds`)
- Press `R` in visual to rotate on the color themes. (see `scr.randpal`)
- Fix some `asm.spacy` and `asm.flgoff` glitches
- Add `ecn` and use it from VR with `scr.randpal`
- `asm.fcnsign` is now working for non-windows binaries
- `asm.(symbol|section)[.col]`
- Added m68k parse pseudo plugin and enhance the arm one
- Fix ROR/ROL ESIL expressions for x86-64 capstone
- Honor `fcn.fcnlines` in fcnvarlist

WebUI
-----
- WebUI moved to a separate repository.
- some of them accessible via r2pm (`r2pm -i www-t www-m`)
- use [Grunt]( http://gruntjs.com/ ), update all dependencies, indent code, minify, ..
- Fixed some XSS vulns
- Added `http.referer` checks to fix CSRF vuln

Architectures
------------
- z80: better analysis
- SNES: better analysis too and support 16-bit immediate operands
- m68k: fixed bugs and improved analysis. honor asm.cpu
- ARM (better analysis and emulation, handle IT)
  - Honor ARM conditional bits to skip bxeq lr and such
  - Better Thumb support
  - Assemble `blx` for arm32 and thumb
- New plugins!
  - Adding initial support for PIC18C diassembler
  - python bytecode disassembler
  - Flash bytecode disassembler

File formats
------------
- PE parser is much better now! (version info + handling even more fucked'up PEs) 
- Support Swift-Demangle
- JSON output for classes+ methods
- Add support for parsing TLS and add TLS callback addresses to the list of entry points
- Extracting iOS's dyldcache is working again (thanks @alvaro_fe)
- *.pyc file format
- *.swf file format
- Better parsing of PE and ELF files
- Add versioninfo support for PE and ELF
- Fix #2780 havecode field

Graphs
------

- Disassemble first basic block in callgraphs
- Summary graph (af;VVP')
- Add graph.gv variables to set custom graphviz styles
- Fix #4374 - ags command to show simplified flowgraph


Bindiffing
----------
- `radiff2 -C` does not analyze by default, mimics `r2 -A`
- Does not diff strings because they are not functions
- Increase memory limit for code diffing

Analysis
--------

- Colorful entropy bars
- file.analyze is only running when the binary contains code
- new `aex` command to emulate an hexpair of native code
- huge improvements for x86 and arm
- Set anal.autoname by default for now
- Adding verbosity in `aaa`
- Improve mips string reference detection with ESIL
- Honor anal.strings in `aae`
- Fix `aap` for static and make it work in debugger
- Find more string references for MIPS and remove some false positives.
- ROP search find honor search.align and detects more cases
- Do not autoname functions by default. Add e anal.autoname
- analysis is deeper than ever: new `aaaa` command
- `aai` command to show analysis statistics info
- `aav` command to show all references for section/map
- added lodsb,stosb and did some rep cosmetic to esil x86
- Initial support for unions
- Redesign the `t` command and add a lot of tests (@oddcoder)
- Initialize BP register in aeim (handy for arm)

Debugging
---------
- New bochs plugin works on Linux, Mac and Windows.
- Coredump generation for Mach0 binaries on iOS and OSX
- MACH0 Coredump loading
- `r2 -d gdb://` no need for `-D gdb`
- Added drw/arw command
- Add r2 -R as alias for dbg.profile
- Alias `doo` for `ood` command 

iOS
---
- Implement [ios9 pangu's tfp0]( http://en.pangu.io/ ) in the debugger
- dyldcache extract
- coredump generation and loading
- Support swift-demangle if found in $PATH


Various changes
---------------
- New r2r program in `radare2-regressions` repo
- Work in progress support for [squashfs]( https://en.wikipedia.org/wiki/SquashFS )
- An `aaaaaa` command
- Add `r_lang_rust`
- Implement `rasm2 -A` to replace `ranal2`
- `rax2 -B` and `-b`
- Handle `~/.config/radare2/radare2rc.d`

Commands
--------
- Extending `wo*` commands to use clipboard
- `Ps` and `PS` commands to save project
- Implement `Cz` like `Cs` with automatic length detection
- Implement new commands: `yl`, `yw`, `ywx`, `wz`
- Implement `ys` to show clipboard as string
- Honor `q` in scripts to stop interpreter
- Add rarun2 execve to avoid posix_spawn

