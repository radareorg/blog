+++
date = "2014-08-28T00:35:56+02:00"
draft = false
title = "Visual mode"
slug = "visual-mode"
aliases = [
	"visual-mode"
]
+++
One of the main complain we get about radare2 is that it has no [GUI]( https://en.wikipedia.org/wiki/Graphical_user_interface ). Maybe we'll get one someday,  but for now, if you don't like the [CLI]( https://en.wikipedia.org/wiki/Command-line_interface ), you can use the *visual mode*, by entering `V`.

Like with very command in r2, you can get help with the `?`. Also, notice the fact that the CLI-command to get the same result it displayed on the top of your terminal.

![help](/blog/images/help.png)

The visual mode comes in several fashion that you can cycle with the `p` key:

- `hex`, the hexadecimal view
- `disasm`, the disassembly listing
- `debug`, the debugger
- `words`, the word-hexidecimal view
- `buf`, the C-formatted buffer
- `annotated`, the annotated hexdump.

The mode can be really powerful when used during dynamic analysis, since it can display the stack, registers state and the disassembly listing in the same time. Use `.` to seek to _eip_, and `Enter` to follow address of the current jump/call.

![visual debugger](/blog/images/visual_dbg-1.png)

If you're not used to radare2, you can use the HUD to quickly access your favorites commands by pressing the `_` key. Just type some letters of your command, and press enter when it gets selected. You can even add *your* commands by edition the hud file in your `R2HOME` folder (Likely equal to `~/.config/radare2/hud`).

![HUD](/blog/images/hud.png)

Like in [IDA]( https://www.hex-rays.com/products/ida/index.shtml ), you can add comments with the `;` key, toggl breakpoints with `F2`, single-step with `F7`, step-over with `F8` and continue with `F9`.

You can also change the type of the current position with the `d` key, to define it as a string, data, code, a function, or simply to undefine it. For example: to rename a function just type 'dr' and then the new name. Othervise, you could hit `v` to get into the *visual code analysis menu* to edit/look closely at the current function.

The 'c' key toggles the cursor mode, which allows to select, copy, modify, insert new strings, assembler or hexpairs depending on the current view and selected column (use `<tab>` in the hexdump view to toggle between hex and strings columns).

There are some hidden pearls inside the Visual mode that allow you to configure the eval variables with the `e` key, or generate an ascii-art basic-block graph of the current function with `V`.

To jump around metadata like functions or strings, you can hit the `t` key, to select where you want to go. Cross-references (also known as XREF) and references are available respectively with the `X` and the `x` key.

![xref](/blog/images/xref.png)
Notice the fact that the main function is detected (top of the picture) as being called only once, by the `entry0` function, which is the entrypoint of the binary.

Since most of the radare2 [developers]( https://github.com/radare/radare2/pulse/monthly ) are using [vim]( http://www.vim.org/ ), you can find most of its shortcuts in visual mode, like `hjkl` to move around, `g` and `G` to go to the begining or the end of the file, `i` to insert, ...

And if you're lost, you can type *classic* r2 command with `:`, or if you're really scared, exit the visual mode with `q`.

All this information and more you can get also if press `?` in visual mode.