+++
date = "1970-01-01T01:00:00+01:00"
draft = true
title = "FLIRTing with radare2"
slug = "flirting-in-master"
aliases = [
	"flirting-in-master"
]
+++
This was one of the most asked features, and thanks to [jfrankowski]( http://failhard.org/pages/about-me.html ) and his RSoC, it's finally here: You can use the [FLIRT]( https://www.hex-rays.com/products/ida/tech/flirt/in_depth.shtml ) technology within radare2!

The goal of FLIRT is to identify (known) functions, relying on a signature system. And since we're a Free software, the (documented) code is [available]( https://github.com/radare/radare2/blob/master/libr/anal/flirt.c ) to everyone!

As usual, the commands are documented *within* radare2, with the `?` char:

```
[0x004013e2]> z?
|Usage: z[abcp/*-] [arg]Zignatures
| z                    show status of zignatures
| z*                   display all zignatures
| z-namespace          Unload zignatures in namespace
| z-*                  unload all zignatures
| z/[ini] [end]        search zignatures between these regions
| za ...               define new zignature for analysis
| zb name bytes        define zignature for bytes
| zc @ fcn.foo         flag signature if matching (.zc@@fcn)
| zf name fmt          define function zignature (fast/slow, args, types)
| zF file              Open a flirt signature file and scan opened file
| zFd file             Dump a flirt signature
| zg namespace [file]  Generate zignatures for current file
| zh name bytes        define function header zignature
| zn namespace         Define namespace for following zignatures (until zn-)
| zn                   Display current namespace
| zn-                  Unset namespace
| NOTE:                bytes can contain '.' (dots) to specify a binary mask
[0x004013e2]> 
```

So, you can use `zF` to open a *F*LIRT database, and `zFd` to *d*ump it.

[Here]( https://asciinema.org/a/12148 ) is an *asciinema* showing the complete walkthrough of importing and using a FLIRT database within radare2.

This should greatly speed up the reversing of unknown binaries. On a side note, we also greatly improved the analysis performance and function detection/naming. Now fire radare2, and reverse some stuff!