+++
date = "2014-05-05T04:15:37+02:00"
draft = false
title = "Initial ascii-art graph layout"
slug = "initial-ascii-art-graph-layout"
aliases = [
	"initial-ascii-art-graph-layout"
]
+++
Lately, a lot of buzz has been going on with the new graph viewer implemented on top of *RConsCanvas*, which renders the basic blocks graph of a function using ascii art.

Today we get an initial layout implemented by pancake which is just a PoC. It's an initial work to implement the proper layouting algorithm to make the graph look more natural and readable by humans.

Additionally the `?` key in `VV` (visual graph) view now shows the help message explaining how to:

- scroll the canvas: `wasd`
- move: `hjkl/HJKL`
- select nodes: `tab/j/f`

Here is a cool screenshot:

![img](/blog/images/VeCMo1r.png)