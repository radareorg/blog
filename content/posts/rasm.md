+++
date = "2016-04-26T11:47:13+02:00"
draft = false
title = "Using RAsm"
slug = "rasm"
+++
RAsm
====

Recently I have noticed that many people gets saturated by the amount of stuff r2 can do and most users end up not learning anything.

So that's why I am going to start writing a series of blog posts showing one feature at a time, making it as simple as possible.

What's rasm?
------------

The 'r' in `r_asm` stands for radare, and it's one of the several libraries shipped with radare2.

As long as it's been designed with modularity in mind, you can use any of the r2 libraries by separate without having to ship or link with the entire framework.

The **RAsm** API is described in the [vapi docs](http://radare.org/vdoc/libr/Radare.RAsm.html) but you may find the *real* one in [the .h file](https://github.com/radare/radare2/blob/master/libr/include/r_asm.h)

Pluggable
---------

Thanks to the modular design, every library shipped in r2 can be extended with plugins.

The plugins can be statically linked inside the library at compile time or loaded from separate shared libraries (dll, dylib, so) when placed in some directories or loaded with the r2 plugin API.

We will not go that far for now, but we must learn that every program in r2 handles the `-L` flag in order to enumerate all the related plugins:

* **r2 -L** : list **IO** plugins
* **rasm2 -L** : list **ASM/ANAL** plugins
* **rabin2 -L** : list **RBIN** plugins
* **rahash2 -L** : list **CRYPTO/HASH** plugins
* ...


Here's the current output for `rasm2 -L`, which shows the following:

* **a** : capable to assemble (ollyasm, custom, ..)
* **d** : disassemble (using capstone, gnu, custom, ..)
* **A** : analyze (associated r_anal plugin)
* **e** : emulate (provides ESIL)

```
$ rasm2 -L
_dAe  8 16       6502        LGPL3   6502/NES/C64/Tamagotchi/T-1000 CPU
_dA_  8          8051        PD      8051 Intel CPU
_dA_  16 32      arc         GPL3    Argonaut RISC Core
a___  16 32 64   arm.as      LGPL3   as ARM Assembler (use ARM_AS environment)
adAe  16 32 64   arm         BSD     Capstone ARM disassembler
_dA_  16 32 64   arm.gnu     GPL3    Acorn RISC Machine CPU
_d__  16 32      arm.winedbg LGPL2   WineDBG's ARM disassembler
adAe  8 16       avr         GPL     AVR Atmel
adAe  16 32 64   bf          LGPL3   Brainfuck
_dA_  16         cr16        LGPL3   cr16 disassembly plugin
_dA_  32         cris        GPL3    Axis Communications 32-bit embedded processor
_dA_  16         csr         PD      Cambridge Silicon Radio (CSR)
adA_  32 64      dalvik      LGPL3   AndroidVM Dalvik
ad__  16         dcpu16      PD      Mojang's DCPU-16
_dA_  32 64      ebc         LGPL3   EFI Bytecode
adAe  16         gb          LGPL3   GameBoy(TM) (z80-like)
_dAe  16         h8300       LGPL3   H8/300 disassembly plugin
_d__  32         hppa        GPL3    HP PA-RISC
_dAe             i4004       LGPL3   Intel 4004 microprocessor
_dA_  8          i8080       BSD     Intel 8080 CPU
adA_  32         java        Apache  Java bytecode
_d__  32         lanai       GPL3    LANAI
_d__  8          lh5801      LGPL3   SHARP LH5801 disassembler
_d__  32         lm32        BSD     disassembly plugin for Lattice Micro 32 ISA
_dA_  16 32      m68k        BSD     Motorola 68000
_dA_  32         m68k.cs     BSD     Capstone M68K disassembler
_dA_  32         malbolge    LGPL3   Malbolge Ternary VM
_d__  16         mcs96       LGPL3   condrets car
adAe  16 32 64   mips        BSD     Capstone MIPS disassembler
adAe  32 64      mips.gnu    GPL3    MIPS CPU
_dA_  16         msp430      LGPL3   msp430 disassembly plugin
_dA_  32         nios2       GPL3    NIOS II Embedded Processor
_dAe  8          pic18c      LGPL3   pic18c disassembler
_dA_  32 64      ppc         BSD     Capstone PowerPC disassembler
_dA_  32 64      ppc.gnu     GPL3    PowerPC
ad__             rar         LGPL3   RAR VM
_dA_  32 64      riscv       GPL     RISC-V
_dA_  32         sh          GPL3    SuperH-4 CPU
_dA_  8 16       snes        LGPL3   SuperNES CPU
_dAe  32 64      sparc       BSD     Capstone SPARC disassembler
_dA_  32 64      sparc.gnu   GPL3    Scalable Processor Architecture
_d__  16         spc700      LGPL3   spc700, snes' sound-chip
_d__  32         sysz        BSD     SystemZ CPU disassembler
_dA_  32         tms320      LGPLv3  TMS320 DSP family
_d__  32         tricore     GPL3    Siemens TriCore CPU
_dAe  32         v810        LGPL3   V810 disassembly plugin
_dA_  32         v850        LGPL3   v850 disassembly plugin
_dAe  8 32       vax         GPL     VAX
_dA_  32         ws          LGPL3   Whitespace esotheric VM
a___  16 32 64   x86.as      LGPL3   Intel X86 GNU Assembler
_dAe  16 32 64   x86         BSD     Capstone X86 disassembler
a___  16 32 64   x86.nasm    LGPL3   X86 nasm assembler
a___  16 32 64   x86.nz      LGPL3   x86 handmade assembler
ad__  32         x86.olly    GPL2    OllyDBG X86 disassembler
a___  32         x86.tab     LGPL3   x86 table lookup assembler
_dAe  16 32 64   x86.udis    BSD     udis86 x86-16,32,64
_dA_  32         xcore       BSD     Capstone XCore disassembler
_dAe  32         xtensa      GPL3    XTensa CPU
adA_  8          z80         NC-GPL2 Zilog Z80
_d__  8          z80.cr      LGPL    Zilog Z80
_d__  32         swf         LGPL3   SWF
_d__  32         propeller   LGPL3   propeller disassembly plugin
```

What can RAsm do for me?
------------------------

Using `rasm2` we can directly use that functionality and be able to assemble and disassemble code from the commandline.

We can provide different options to it, which defaults to the current system ones:

* **-a** : specify plugin arch name  (arm, x86, mips, ...)
* **-b** : specify bits (16, 32, 64) (*arm-16 is thumb*)
* **-o** : offset where this instruction is
* **-s** : change syntax for ATT, MASM, INTEL, ..
* **-e** : swap endian (for bi-endian archs)


By default `rasm2` will assemble the argument passed, but using the **-d** switch it will expect an hexpair and give back the disassembled instruction for the selected architecture.

Rasm2 is not just an instruction assembler/disassembler. It can also handle source files like `nasm` or `gas` do. As well as disassemble raw files without parsing binary headers using the **-f** option.

Commandline
-----------

Let's see how this thing works:

```
$ rasm2 -a x86 -b 32 "mov eax, 33"
b821000000
$ rasm2 -a x86 -b 32 -d b821000000
mov eax, 33
```

In case the default assembler doesn't supports a specific instruction we can switch to another one:

```
$ rasm2 -L | grep x86 | grep ^a
a___  16 32 64   x86.as      LGPL3   Intel X86 GNU Assembler
a___  16 32 64   x86.nasm    LGPL3   X86 nasm assembler
a___  16 32 64   x86.nz      LGPL3   x86 handmade assembler
ad__  32         x86.olly    GPL2    OllyDBG X86 disassembler
a___  32         x86.tab     LGPL3   x86 table lookup assembler
```

like this:
```
$ rasm2 -a x86 -b32 'jbe 33'
Cannot assemble 'jbe 33' at line 3

$ rasm2 -a x86.nasm -b32 'jbe 33'
0f861b000000
```


Using the API
-------------

As long as this functionality is implemented in a library we can just use the API from C, C++, or any of the languages supported by [radare2-bindings](https://github.com/radare/radare2-bindings):

```c
#include <r_asm.h>

int main() {
	RAsmOp op = {0};
	RAsm *a = r_asm_new ();
	r_asm_use (a, "x86");
	r_asm_set_bits (a, 32);
	r_asm_set_pc (a, 0x80000);
	r_asm_disassemble (a, &op, "\x90", 1);
	printf ("%s\n", op.buf_asm);
	r_asm_free (a);
	return 0;
}
```

The latest version of r2 supports a simplified API which can be used like this:

```c
#include <r_asm.h>

int main() {
	RAsm *a = r_asm_new ();
	r_asm_set_arch (a, "x86", 32);
	
	/* assemble */
	int i, buflen;
	ut8 *buf = r_asm_from_string (a, 0x4000, "nop", &buflen);
	if (buf) {
		for (i = 0; i < buflen; i++) {
			printf ("%02x", buf[i]);
		}
		printf ("\n");
	}
	
	/* disassemble */
	char *code = r_asm_to_string (a, 0x4000, "\x90\x90", 2);
	if (code) {
		printf ("%s\n", code);
		free (code);
	}
	r_asm_free (a);
}
```

Scripting
---------

The [radare2-bindings](https://github.com/radare/radare2-bindings) repository provides several bindings for a long list of programming languages.

Those bindings are automatically generated by [valabind](https://github.com/radare/valabind), which parses the VAPI api description files and generates the wrapper code for Python, NodeJS, Ruby, C#, ...

Check out the examples under the `t/` subdirectories to find out examples like this Python one:

```python
from r2.r_asm import *

def ass(a, arch, op):
	print "OPCODE: %s"%op
	a.use (arch)
	print "ARCH: %s"%arch
	code = a.massemble (op)
	if code is None:
		print "HEX: Cannot assemble opcode"
	else:
		print "HEX: %s"%code.buf_hex

a = RAsm()
ass (a, 'x86.olly', 'mov eax, 33')
ass (a, 'java', 'bipush 33')
```

R2Pipe
------

R2Pipe provides a simple `command` -> `result` interface to talk to **radare2**, this eases the integration of radare2 into several other programs, without depending on a local installation of r2 because r2pipe can speak to `http`, `pipes` and other channels.

For this we must use the proper r2 commands:

* **pa** assemble code inline
* **pad** disassemble code inline

```
[0x00000000]> pa nop
90
[0x00000000]> pad 90
nop
```

We can also temporary change the arch/bits setup using the `@` operator like this (see `?@?` output for more help):

```
[0x00000000]> pad 0d20a0e3 @a:arm:16
movs r0, 0xd
b 0x746
```

And merge this stuff into something like this nodejs script:

```javascript
'use strict';

const r2pipe = require('r2pipe')
r2pipe.connect('http://cloud.radare.org/cmd', (r2p) {
	r2p.cmd("pad 90@a:x86:32", (res)=> {
		console.log('0x90 for x86:32 means:', res);
	})
})

```


Conclusions
------------

Radare2 provides multiple functionalities splitted into separate libraries, which at the same time are extended by plugins (for r_asm using capstone, keystone, nasm, GNU binutils, ..).

This functionality is accessible thru the r2 commandline, the rasm2 program and using the C API.

[--pancake](https://twitter.com/trufae)
