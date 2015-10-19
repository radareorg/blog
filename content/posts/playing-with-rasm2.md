+++
date = "2014-05-02T12:43:49+02:00"
draft = false
title = "Playing with rasm2"
slug = "playing-with-rasm2"
aliases = [
	"playing-with-rasm2"
]
+++
Radare2's assembler/disassembler is `rasm2`, and albeit being used internally, it is also a standalone binary that you can use.

It can of course disassemble
```
$ rasm2 -d 89d85d90
mov eax, ebx;pop ebp;nop
```
but also assemble
```
$ rasm2 'mov eax, ebx;pop ebp;nop'
89d85d90
```

Not only x86, but also mips
```
$ rasm2 -a mips  'addiu a1, a2, 8'
0800c524
$ rasm2 -a mips -d 0800c524
addiu a1, a2, 8
```

and many more. You can have the full list with `rasm2 -L`.

It can also describes all opcodes, in case you don't know them all.
```
$ rasm2 -w sqrtpd
compute square roots of packed double-fp values
```

There is a [RSoC]( http://rada.re/rsoc ) task about adding a new architecture, and we are always looking for new related [unit-tests]( https://github.com/radare/radare2-regressions/tree/master/t.asm ).