+++
date = "2018-05-31T01:45:16+02:00"
draft = false
title = "GSoC'18 Progress Report - May"
slug = "gsoc_2018_progress_report_may"
aliases = [
	"gsoc 2018 progress report may"
]
+++

<script type="text/javascript" src="https://cdnjs.cloudflare.com/ajax/libs/mathjax/2.7.1/MathJax.js?config=TeX-AMS-MML_HTMLorMML"></script>

A few weeks passed but our students worked hard and achieved a tremendous results already. Lets see what they have to say about this:

## Cyanpencil's update

Hi, I'm [cyanpencil](https://www.github.com/cyanpencil), and in those initial three weeks of the GSoC 2018 I took care of the commands relative to graph drawing (all the commands starting with `ag`).

Those commands were a bit confusing because each of them required a different syntax / used different config variables, resulting in a `ag?` help are mixed and hard to understand.

After some discussion with the rest of the community (special thanks to [@pancake](https://twitter.com/trufae) and [@XVilka](https://github.com/XVilka) who followed me closely during this process), we came up  with a standard syntax for this family of commands:

**`ag<graph type><output format>`**

For example, if I now want to draw the callgraph in interactive ascii view, I just have to type `agcv`. For the callgraph in tiny ascii art, it's `agct`, etc..


I also added a few new graph types (the import graph and the data graph) and worked hard to make all graph types compatible with every output format. (The only exception is the diff graph, that currently supports only `d` and `w` formats)

During this process, I had to take care [of](https://github.com/radare/radare2/issues/8385) [some](https://github.com/radare/radare2/issues/9716) [issues](https://github.com/radare/radare2/issues/9962) [related](https://github.com/radare/radare2/issues/6220) [to](https://github.com/radare/radare2/issues/9199) [graphs](https://github.com/radare/radare2/issues/9867) as well.

All of this resulted in a new, better `ag?` help that we hope clear even for the new users of r2:

```
|Usage: ag<graphtype><format> [addr]
| Graph commands:
| agc[format] [fcn addr]  Function callgraph
| agf[format] [fcn addr]  Basic blocks function graph
| agx[format] [addr]      Cross references graph
| agr[format] [fcn addr]  References graph
| aga[format] [fcn addr]  Data references graph
| agd[format] [fcn addr]  Diff graph
| agi[format]             Imports graph
| agC[format]             Global callgraph
| agR[format]             Global references graph
| agA[format]             Global data references graph
| agg[format]             Custom graph
| ag-                     Clear the custom graph
| agn[?] title body       Add a node to the custom graph
| age[?] title1 title2    Add an edge to the custom graph
|
| Output formats:
| <blank>                 Ascii art
| v                       Interactive ascii art
| t                       Tiny ascii art
| d                       Graphviz dot
| j                       json ('J' for formatted disassembly)
| g                       Graph Modelling Language (gml)
| k                       SDB key-value
| *                       r2 commands
| w                       Web/image (see graph.extension and graph.web)
```

## sivaramaaa

Here are the few highlights of my work for this month :

### afta optimization

I started first trying to optimize afta which was before emulating entire function, refactored it to buffer only a few instructions before call instruction and emulate it when we hit a known function call

### Improve variable recovery

Previously function stack frame was calculated at the end of the analysis, this could be wrong since stack frame is not fixed, so I tweaked the analysis loop to fix this issue

### Working with types commands

1. Array of struct now works with ts command

```
[0x00000000]> "td struct foo { int a; int b; };"
[0x00000000]> "td struct bar { int x; struct foo y[2]; };"
[0x00000000]> ts bar
  pf d[2]? x (foo)y
[0x00000000]> .ts bar
  x : 0x00000000 = 0
  y :
    [
                  struct<foo>
       a : 0x00000004 = 0
       b : 0x00000008 = 0
                  struct<foo>
       a : 0x0000000c = 0
       b : 0x00000010 = 0
    ]
```

2. Manage enum types properly - enum data type is now stored in sdb in a consistent way with rest of the type storage,

3. Fix te to list all values of enum

```
[0x00000000]> "td enum Foo {COW=1,BAR=2};"
[0x00000000]> te Foo
COW = 0x1
BAR = 0x2
```

4. Fixed struct offset for dst operand in ta

5. Currently working on structure offset propagation, which requires a lot of change in opcode parser to support RAnalop.dst/src in all architectures, I am going to finish this soon and move onto Type Inference task!

## Kriw

I have implemented C-like internal representation so as to help decompilation process (e.g. structuring CFG). For this the [SimpleCAST](https://github.com/radareorg/radeco-lib/pull/145) was implemented. Basically SimpleCAST is a CFG of C-like statements. For checking the correctness I wrote converter from SimpleCAST to existing CAST and unit tests for it.

The next step was the [recovering statements from IR](https://github.com/radareorg/radeco-lib/pull/162), and prototyping it by implementing required functions and data strictures [read some papers](https://github.com/radareorg/radeco-lib/issues/156).

For the next week I plan to work on those things:

- implement converter from IR to SimpleCAST
    - collect local variables of target function from r2api
    - recover expressions from SSA
    - recover statements (assignment, function call, goto, etc)
- refine SimpleCAST

## HMPerson1

The most important changes this week are the elimination of comments, [moving everything related to registers handling in edges](https://github.com/radareorg/radeco-lib/pull/157) and [register API migration](https://github.com/radareorg/radeco-lib/pull/161)

Apart from this I studied thoroughly the [No More GOTOs](https://net.cs.uni-bonn.de/fileadmin/ag/martini/Staff/yakdan/dream_ndss2015.pdf) paper and extracted algorithm from it:

1. perform DFS on CFG to find backedges; record post-order trace
    - record trace b/c we'll modify the CFG later
2. for each node $n$ in trace:
    1. if no backedge points at $n$:
        1. find set of nodes dominated by $n$
        2. if region has single successor $s$:
            1. structure acyclic SESE region $(n,s)$
    2. else: (loop)
        1. compute initial loop nodes from given latching nodes
        2. find abnormal entries and "funnel" them into a single entry
        3. compute initial exit nodes $N_{succ} = \text{succ}(N_{loop}) \setminus N_{loop}$
        4. refine $N_{loop}$ and $N_{succ}$
        5. if $\left\|N_{succ}\right\| > 0$:
            1. if $\left\|N_{succ}\right\| = 1$:
                1. $\{s\} = N_{succ}$
            2. else:
            	1. pick $s \in N_{succ}$ with the smallest post-order visit time
            	2. "funnel" abnormal exits into $s$
            3. replace all edges to $s$ with `break`
        6. insert a "loop continue" node $c$ (not in the paper but I can't think of any other way to do this)
        7. retarget each backedge to $c$
        8. structure acyclic SESE region $(n,c)$
        9. remove $c$
        10. put the AST node inside an infinite loop
        11. refine AST

As a requirement for this algorithm I did the initial preparations and [implemented](https://github.com/radareorg/radeco-lib/pull/160):
- Some graph algorithms and tests for them
- High-level structuring algorithm
- Basic acyclic region structuring (no refinements yet)
- Conditions are arena-allocated now to avoid excessive cloning

My next plans are:
- Implement the parts involving loops
- Improve completeness and correctness; no refinements yet
- Write QuickCheck tests for the whole algorithm just to make sure it doesn't crash

## mandlebro

During the first two weeks of GSOC, I worked on a couple of necessary features before starting working on the debugging aspects of [Cutter](https://github.com/radareorg/cutter):

 * **independent seeks**: this allows different widgets to have different seeks. Previously, all widgets would always be in sync with each other, and with the radare2 instance running in the background. Now, you can toggle if you would like your widget to be in sync with the other widgets.

![indep_seek](https://raw.githubusercontent.com/fcasal/fcasal.github.io/master/syncseek.gif)

 * **multiple widgets**: this allows users to open more than one instance of the same widget. This means that you can now look at several  functions in graph/linear mode at the same time via the menu `Windows`->`Add extra...`:

![](https://raw.githubusercontent.com/fcasal/fcasal.github.io/master/crop_mult_widget.png)

![](https://raw.githubusercontent.com/fcasal/fcasal.github.io/master/mult_graph_crop.png)

Both these features were already [merged](https://github.com/radareorg/cutter/commit/0cea9e3287fa12a8a240834a7484c1a957f686c1) and you can try them on the master branch of [Cutter](https://github.com/radareorg/cutter)!

Currently I am working on widgets that show debug information:
 * value of registers
 * stack addresses and telescoping
 * backtrace information
 * create a general debug window which integrates all these widgets
