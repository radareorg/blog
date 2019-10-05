+++
date = "1970-01-01T01:00:00+01:00"
draft = true
title = "Reverse BIOS/UEFI with Radare2"
slug = "reverse-biosuefi-with-radare2"
aliases = [
	"reverse-biosuefi-with-radare2"
]
+++

* R2 Script .r2
* Small drawing of composition of UEFI
* TE/PE File (pf) -> http://radare.today/parsing-a-fileformat-with-radare2/
* Usage of to/tf/tl/t -> http://radare.today/types/

# Extracting the BIOS/UEFI

* Flashrom http://flashrom.org/Flashrom // Coreboot http://www.coreboot.org/
* UEFITools https://github.com/LongSoft/UEFITool // Bios\_extract http://www.coreboot.org/Bios_extract

# Binary patching

In order to modify a binary, radare2 must open the file in a writing mode using `-w` flag: `r2 -w binary.exe`

### Using CLI mode

Using seek `s` to move around the binary, wx to insert/write some hexadecimal: `wx 9090` for example, writes two `nop` and `wa` to assemble and write the opcodes: `wa push ebp` for example.

![wx and wa](/blog/images/wxwa-1.png)

### Using Visual mode

The Visual Mode is the best way to patch assembly or hexadecimal. It is similar to Hexedit in usage but with much more functionalities. To enter the Visual mode `V`. You can navigate between the various panel using `p` to move to the next panel and `P` to the previous panel.

#### Hexadecimal patching

First let's patch the binary using hexadecimal patching. First go to the Visual hexadecimal panel:

![Visual Mode](/blog/images/V.png)

Then activate the cursor selection: `c`. You can now select hexadecimal or ASCII representation using `tab` (notice the white border).

![Cursor mode](/blog/images/Vc.png)

You can now insert hexadecimal code or string by typing `i`.

![Insert String](/blog/images/Vcadd.png)

#### Opcode patching and Visual assembler

Now let switch to the disassembly panel in visual mode (Vpp).

![Visual Mode: Disassembly mode](/blog/images/VisualDisassembly.png)

Like the hexadecimal view, you can enable the cursor mode using `c` and select opcodes you want to replace.

![Cursor mode in Disassembly](/blog/images/Vcc.png)

To change the `push ebp` for example by `xor eax, eax` opcode you can use `a`:

![Va](/blog/images/Va.png)

Visual mode also contain an interesting live preview Opcode patching: the Visual Assembler `A`.

Let's replace this `xor eax,eax` by a `jmp 0x08048376`, notice the arrow on the left (==Live preview==).

![Visual Assembler](/blog/images/VA.png)