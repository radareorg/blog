+++
date = "1970-01-01T01:00:00+01:00"
draft = true
title = "Manually analysing binaries"
slug = "manually-analysing-binaries"
aliases = [
	"manually-analysing-binaries"
]
+++
Most of the time, when reversing binaries with radare2, the first command you issue is `aa`, to **a**nalyse **a**ll. But maybe this time, your binary is too convoluted, and radare2 doesn't detect some functions. Or maybe you're opening a *really big* binary, and the analysis is taking too much time. Or maybe you just found a bug ;)

By default, radare2 is doing a recursive analysis with a configurable depth (`anal.depth`). Currently, it's not perfect, and it may miss some functions.

You can define a function with `af`, this will launch the analysis engine on the current offset, defining a function, and *recursively* analyse from here.

If your function is really convoluted, you can force its size with the `af+ offset size name` (and please open a bug ;).

Radare2 is also able, with `ap`, to detect some functions by searching for [preludes]( https://en.wikipedia.org/wiki/Function_prologue ) and analysing them. This is a bit hackish, since it's easy for an adversary to write false preludes. Use it with caution, or you may end up with several thousands of fake functions in you database.

### XREF
You can manually add (and remove) cross-references with the following commands:

- `axc offset`, to add a *jump* reference
- `axC offset`, to add a *call* reference
- `axd offset`, to add a *data* reference

### Type changing
If radare2 is unable to correcly detect the type of the current offset, you can change it with:

- `Cf [size]`, to define current offset to *size* bytes as a formatted struct (see `pf?`)
- `Cd [size]` to define as data
- `C- [size]` to undefine stuff, *aka*, define as code
- `Cs [size]` to define as string

TODO:

- ESIL
- Tracing
- Hints (`ah`)