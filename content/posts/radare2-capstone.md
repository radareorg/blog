+++
date = "2017-03-09T18:30:00+01:00"
draft = false
title = "Radare2 and Capstone"
slug = "Radare2-capstone"
aliases = [
    "Radare2-capstone"
]
+++

# Radare2 and Capstone 

This blogpost is the response to an observable fact: **People don't know that Radare2 is using Capstone/Keystone/Unicorn**.

This is also a blogpost to address the numerous comparisons done online to these two different components.

# Defining the tools

* **Capstone** is a multi-architecture disassembly framework

* **Keystone** is a multi-architecture assembler framework

* **Unicorn Engine** is a multi-architecture CPU emulator framework

* **Radare2** is a reverse engineering framework, it includes, in addition to other functionality: multi-architecture disassembly, assembly, and CPU emulation.

Radare2 is using **Capstone** _as its default disassembler since **2014**_ for arm, m68k, MIPS, PowerPC, SPARC, X86 and XCore. 

**Radare2** also allows you to use other disassembly engines such as udis for x86 or gnu implementation for arm, MIPS, PowerPC and SPARC. 

**Radare2** also contains many other architectures that are not supported by Capstone, such as 6502, VAX, RISC-V... (Full list can be obtained using `rasm2 -L`)

You can use **Keystone** within **Radare2** by installing the Keystone plugin via r2pm, the radare2 package manager.

`$> r2pm -i keystone-lib && r2pm -i keystone`

Similarly you can use Unicorn Engine within Radare2 by installing the Unicorn Engine plugin via r2pm:

`$> r2pm -i unicorn-lib && r2pm -i unicorn`

![unicorn Engine in radare2](https://pbs.twimg.com/media/CNVxHanWIAA5WeL.png)

Both of these integrations were available prior to the public release of Keystone and Unicorn Engine.

## Why the current comparisons between Radare2 and Capstone are idiotic

So now here are two questions you should ask yourself:

* Would you compare a set of wheels with a car ?
* Would you compare a library with the software that uses it ?

This is how these dumb comparisons would actually look like given all those elements:

|   | Radare2 | Capstone | Keystone | Unicorn-Engine |
| --- | --- | --- | --- | --- |
| Multi-architecture disassembly framework  | YES | YES | NO | NO |
| Multi-architecture assembler framework | YES | NO | YES | NO |
| multi-architecture CPU emulator framework | YES | NO | NO | YES |

As one can notice, this has zero value, and therefore should not be done

## API

One of the comparisons that is often done is between the APIs. Without referring to the previous paragraph.

For this example I am going to show the Capstone API vs R_asm as already presented [here](http://radare.today/posts/rasm/) vs R2pipe which is the recommended way to script radare2.

### R_ASM

#### Installation

`r2pm -i r2api-python` (Other language available see [here](https://github.com/radare/radare2-bindings))

#### Usage

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

### R2Pipe

#### Installation

`r2pm -i r2pipe-py` or `pip install r2pipe` (Other language available see [here](https://github.com/radare/radare2-r2pipe))

#### Usage

```python
import r2pipe

r2 = r2pipe.open("-")
r2.cmd("e asm.arch=x86")
r2.cmd("e asm.bits=64")
nopHex = r2.cmd("pa nop")
nopCode = r2.cmd("pad 90")
r2.quit()
```

### Capstone API

#### Installation

`pip install capstone`

(you will need keystone for assembly too)

#### Usage

```python
from capstone import *

CODE = b"\x55\x48\x8b\x05\xb8\x13\x00\x00"

md = Cs(CS_ARCH_X86, CS_MODE_64)
for i in md.disasm(CODE, 0x1000):
     print("%s\t%s" %(i.mnemonic, i.op_str))
```


## Conclusion

Comparisons between radare2 and Capstone do not make sense. 

**So stop doing it.**

Secondly why use radare2 for analysis and Capstone API for scripting when you can use radare2 for both tasks without any effort.

The APIs, though different, produce the same results as expected. Radare2 will have the advantage though, as additional functionalities such as analysis (cross-references) and debugging for example can be applied to the data coming from Capstone. Additionally, radare2 also provides an embedded disassembler.

It makes more sense to compare radare2's ESIL engine to the Unicorn Engine, or Udis to Capstone.


