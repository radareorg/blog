+++
date = "2016-06-23T21:00:10+02:00"
draft = false
title = "How goes the Google Summer of Code?"
slug = "gsoc-midterm-2016"
+++

As you know this year we're taking part in the [Google Summer of Code](https://summerofcode.withgoogle.com/), with 3 students on cool tasks:
- completing [radeco]( https://github.com/radare/radeco ), our own decompiler, by [sushant94](https://twitter.com/_sushant94)
- improving variables and function arguments analysis, by [anoddcoder](https://twitter.com/anoddcoder)
- completing our web interface, by [gauthiergc](https://twitter.com/GautierGC)

Lets see what our students were supposed to do, what they did, and what is now planned.

Radeco
------

The main task for GSoC’16 is to introduce [type inference]( https://en.wikipedia.org/wiki/Type_inference ) for radeco and produce pseudo C output. Towards this goal, over the last couple of weeks I’ve:

- Fixed several bugs in ESIL and radeco,
- Refactored radeco to construct [SSA](https://en.wikipedia.org/wiki/Static_single_assignment_form) and [CFG]( https://en.wikipedia.org/wiki/Control_flow_graph ) in one single step (this was done in two separate stages previously), 
- Major improvement in logging coverage inside the library making tracking bugs and mistakes faster,
- Implemented an [LLVM IR](https://en.wikipedia.org/wiki/LLVM#LLVM_intermediate_representation)-like text representation for the internal SSA IR (radeco IR),
- Implemented *find and replace* on radeco IR allowing one to identify and replace common assembly idioms. The find and replace patterns are specified through a [LISP](https://en.wikipedia.org/wiki/Lisp_(programming_language))-like language.
- Implemented [common subexpression elimination](https://en.wikipedia.org/wiki/Common_subexpression_elimination) to reduce the IR
- Added support to load entire binaries and [inter procedural analysis](https://en.wikipedia.org/wiki/Interprocedural_optimization).

Over the remaining duration of GSoC my aim will be to implement pseudo C output and type inference for variables. In doing so, the first major hurdle seems to be in variable identification. Since the quality of analysis by the type inference algorithm is directly tied to accuracy of variable identification, it would be worthwhile to spend some time and implement techniques outlined in the [DIVINE](http://research.cs.wisc.edu/wpis/papers/vmcai07.invited.pdf) paper. As of now, the above have no yet been merged to the master branch of radeco and this will be done shortly!

If you’d like to see the work in progress and try out some of these features, checkout [radeco-lib/cfg_ssa](https://github.com/radare/radeco-lib/tree/cfg_ssa)

— [sushant94](https://twitter.com/_sushant94)

Function arguments and variables
-------------------------------
Hello I am [oddcoder](http://oddcoder.com) from Egypt, for the past couple of months I have been working on improving **Function Argument Detection** (and other related things). In fact I have been using Radare2 for a while before I even knew about the *Google Summer of Code*, and I had some kind of idea what the final product should be. Due to the fact that I never went into the codebase I started with some light tasks like fixing the type storage system. Doing that introduced me to some of technologies in software engineering that I never used before, like [continuous integeration systems](http://ci.radare.org) for example. After that I started extending  function argument storage. Now, radare2 can handle stack pointer and base pointer arguments/vars as well as arguments passed into registers. As a bonus, I also created additional analysis rounds based on ESIL, capable of detecting most (if not all) arguments and variables on the stack, for both stack pointer based and base pointer based things, in an arch independent way!

For the next few days, until the midterm evaluation, I will resolve as much of the issues related to arguments/vars as I can, and maybe implement another analysis round for detecting arguments passed via registers. After the midterm I'll work on type propagation for function arguments, actually I still don't know much about this part so I dont have too much to say here yet :D

In fact, life was not always smooth, being the fist time to participate in a project that is bigger than a couple of lines of code and with all of these eyes watching,  I was a bit nervous and worried. But as time went on I felt relaxed, xvilka and pancake helped me a lot, and spend time when needed, with me, answering my questions. Another major problem was that radare2 uses [sdb](https://github.com/radare/sdb), and I wasn't taught about databases at school. In fact, due to its extensive use, I started to figure out my way around a lots of the problems I faced in other places.

— [anoddcoder](https://twitter.com/anoddcoder)

WebUI
-----

Hi! I'm [Gautier](https://blog.gautiercolajanni.fr) from Montréal, Canada and for this GSoC I'm working on the WebUI. Currently there is several UI available for radare and I'm doing some work to improve the one based on Material Design.

To show my abilities before submitting my proposal, I've worked on the management of the dependencies. By this time, the libraries used were copied into the repository, making it heavy and difficulty maintenable. I've forced the usage of [gulp](http://gulpjs.com/) and [bower](https://bower.io/) with [npm](https://www.npmjs.com/) for each UI and added a build process who produce the UI with all the mandatory dependencies packed in the same directory.

After my acceptation into the program, I've done the following main tasks:

 * Implementing a search function with autocompletion ([article](https://blog.gautiercolajanni.fr/gsoc/2016/06/01/autocompletion-field-implementation-with-vanilla-js.html))
 * Adding sorting capacity on tables ([article](https://blog.gautiercolajanni.fr/gsoc/2016/06/17/datatables.html))
 * Implement dual-column resizable views ([article](https://blog.gautiercolajanni.fr/gsoc/2016/06/21/implement-dual-column-resizable-view.html))

Also, I've worked on several smaller tasks:

 * Adding a "make watch" (gulp watch) to automatically propagate the change during developpment process
 * Writting global files about the UI: readme, all licences and how to contribute
 * Replacing the monospace font to improve readability
 * Fix a bug on the left menu on small screen who doesn't want to close
 * Adding a dialog to inform user of network problem and trying to recover

By this time, I'm currently in progress on several tasks. I'm implementing a better abstraction for the dual-column view through a widget approach. I will also fix some visual glitches on the status bar and improving the integration of the DataTable plugin before propagate on the whole tables.

That's a short resume of what I've done during the firsts week. I will be far more efficient for the rest of the adventure. I've underestimated the load of my course for June but that's over now and I'm truly happy to dedicate myself more extensively into this project! Step by step, I become more common with the code base, the UI framework and r2. I'm more than ever motivated to make great improvements to build a proper WebUI for radare.

For the following weeks, my tasks will concerns at least:

 * Keybinding / Shortcuts
 * Implenting the S output
 * More stuff editable/viewvable/manipulable
 * New HexDump
 * New console (with colors and history)
 * Some considerations to replace the current graph library and build interactives graphs (basic-blocks, functions, bindiff)
 * Improving the script panel
 * Play with T command
 * Making the UI usable on a wide range of devices
 * Cleaning code / documentation

I'll also manage to solve most of the [issues](https://github.com/radare/radare2-webui/issues) provided.

If you want to follow my progress, you can see it through my [fork](https://github.com/gcolajan/radare2-webui) (I'm used to use branches) and also through my [blog](https://blog.gautiercolajanni.fr).

— [gauthiergc](https://twitter.com/GautierGC)
