+++
date = "2014-12-03T10:40:18+01:00"
draft = false
title = "The new web interface"
slug = "the-new-web-interface"
aliases = [
	"the-new-web-interface"
]
+++
Thanks to [pwntester]( https://twitter.com/pwntester ), we've got a new web-interface for radare2! You can either get it by using the latest [git]( https://github.com/radare/radare2 ), or try it on our [cloud]( http://cloud.radare.org/p/ ).

Lets highlight the new features:
    
# Graphing
The web-interface is now using [viz.js]( https://github.com/mdaines/viz.js/ ) to show interractive graphs, and the disassembly has now syntax highlighting, like the command line interface. When we say Interractive, we mean that you can not only move the graph, but also modify, edit and annotate it.
    
![shortcuts](/blog/images/kkjXu31.png)

# IDA shortcuts
Afficionados of IDA shouldn't be lost anymore, since the web-interface now shares a lot of its shortcuts:

- `n` to re*n*ame a function
- `g` to *g*o on a specific offset
- `;` to add a comment
- `c` to define as *c*ode
- `u` to *u*ndefine
- `space` to switch between disassembly and graph view

# New views
You can now see hexdump, graph view, disassembly, settings and strings since each of them has a dedicated webview!

By the way, changes made within the web interface are persistent, this means that you can build your colourscheme in an interractive manner. nOf course, every useful information that can be used to move in the binary can be displayed in the side bar, like functions, sections, imports, relocs, and flags.

![colour selector](/blog/images/SmAmHLD.png)

Since everything is not doable within the web interface (yet), you'll find a command-line widget at the bottom of your screen, and the changes are propagated in real-time.