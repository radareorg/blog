+++
date = "2015-05-14T23:45:49+02:00"
draft = false
title = "Using radare2 to pwn things"
slug = "using-radare2"
aliases = [
	"using-radare2"
]
+++
While more and more people are using radare2 during ctf, in the same time we've got more and more complains that there is not enough documentation about radare2.

This article's goal is to make a small cheat-sheet when it comes to pwn things with our beloved piece of software.

Keep in mind that:

* Every character has a meaning (`w` stands for *write*, `p` stands for *print*, …)
* Every command can be a succession of character (`pdf` stands for `p`: *print*, `d`: *disassemble*, `f`: *function*
* Every command is documented with ? (`pdf?`, `???`, …)

# Search

* search a string in memory/binary: `/ yourstring`
* search for rop gadgets: `/R`
* search for rop gadgets with a regexp: `/R/`
* show strings: `iz`
* find writeable sections: `iS | grep perm=..w`
* find executables sections: `iS | grep perm=...x`
* find xref of a function: `axt [offset|yourfunctioname]`
* list libc imports: `is~imp.`
* generate cyclical pattern: `ragg2 -P $SIZE -r`
* find offset of pattern: `wopO $VALUE`
* change the deep of the rop-search: `e search.roplen = 4`
* computing how far a symbol is from where you currently are: `fd [offset|yourfunctioname]`
* find protections in binary:
  * `i~canary` for canaries
  * `i~pic` for Position Independent Code
  * `i~nx` for non-executable stack

# Emulation
* initialize emulation: `aei`
* deinitialize emulation: `aed`
* emulate a whole function: `aef`
* single-step: `aes`

# Display
* hexdump: `pxw [len] [@ offset]`
* get offset of a symbol: `?v sym.main`
* disassemble a whole function: `pdf @ [offset|yourfunctioname]`
* list functions: `afl`
* get in which function an address is used: `afi address~fcn`
  * `afi` to get function information
  * `~fcn` to grep for "fcn"
  * append `j` to get a JSON output : `ij`, `pdfj`, `/Rj` ...
* calculus of offsets: `? 0x20 + 0x4028a0`

# Debugger
* connecting the gdbserver `r2 -D gdb -d [binary] gdb://[address:port]` ([full doc](https://github.com/radare/radare2/blob/master/doc/gdb))
* connecting the remote windbg `r2 -D wind -d [binary] windbg://[pipe address]` ([full doc](https://github.com/radare/radare2/blob/master/doc/gdb))
* show registers: `dr=`
* emuling strace: `dcs*`
* disassemble at register `reg`: `pd [len] @ [reg]`

# Misc
* emulating socat: `rarun2 program=./plzpwnme.exe listen=4444`
You should really take a look at `rarun2`'s manpage.

Feel free to [tell us]( irc://irc.freenode.net/radare ) if you think that we missed something, and good luck for the [Defcon CTF Quals]( https://2015.legitbs.net/ )!
