+++
date = "2014-05-04T01:33:41+02:00"
draft = false
title = "Jumping around in visual mode"
slug = "jumping-around-in-visual-mode"
aliases = [
	"jumping-around-in-visual-mode"
]
+++
Yesterday, someone asked on [IRC]( irc://irc.freenode.net/radare ) how to jump around in visual mode (`V` key to activate it, and `?` for help, as usual). This is a perfect pretext for another blogpost.

To move in visual mode, you can use:

- `g` to seek to the begining of the file
- `G` to seek to the end of the file
- `hjkl` to move, *à la vim*.
- `mK` to set the mark _K_ at the current offset
- `'K` to seek to the previously set _K_ mark.
- `o offset` to seek the the offset _offset_.
- `Enter` to follow the current call/jump
- `u` and `U` to undo/redo the previous seek, like the `Escape` key in IDA.
- `x` to show *xrefs*, and to seek to whichever you want.
- `c` to toggle the cursor mode
- `>` and `<` to seek aligned on the current block size.
- `.` to seek to current program counter.

As you can see, there are a lot convenient way to move in visual mode!
