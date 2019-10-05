+++
date = "2015-06-15T21:36:21+02:00"
draft = false
title = "Update From the GSoC"
slug = "update-from-the-gsoc"
aliases = [
	"update-from-the-gsoc"
]
+++
As you know, we have 2 students working on r2 for the [Google Summer of Code]( https://developers.google.com/open-source/soc/ )! 

As we're 3 weeks into the Summer, here's what one of our student, [sushant94]( https://github.com/sushant94 ) has to say about what he's been working on!

![GSoC logo](/blog/images/gsoc2015-300x270.jpg)

It's been three weeks into GSoC and I'm having an amazing time. I am working along side [dkreuter](https://twitter.com/dkreuter_) and been learning tons from him too!

[Here](https://github.com/radare/radeco) is the repository where you can track our progress and also give us suggestions :)

We chose [Rust](http://www.rust-lang.org/) as our language for implementation. Though at first I was a bit scared of this choice, I quickly realized how great the language is! Rust has allowed be to far more productive, after of course my initial battles with the borrow checker.

The zero-cost abstractions allowed by Rust has been a great so far!

This is a quick roundup of what I've been upto:

* Firstly, I got an [ESIL](https://github.com/radare/radare2/wiki/ESIL) parser up and running. We use this parser to convert to an [IR]( https://en.wikipedia.org/wiki/Intermediate_language ) which is easier to perform static analysis on. While the ESIL is amazing for emulation purposes, it's not so much for static analysis as it has a very large number of supported opcodes. The *RadecoIR* (name subject to change) is much more simplified in terms of the number of opcodes. Also, ESIL is primarily just strings (as suggested by its name "Evaluable Strings Intermediate Language") which could be pretty hard to work with.
* The next step was to build a Control Flow Graph ([CFG](https://en.wikipedia.org/wiki/Control_flow_graph)) out of the IR. Building a control flow graph will allow us to reason about the way the control flows in the program, and hence, helps us better understand the different programming constructs that go into making it. To make debugging and visualizations easier, I first went ahead and implemented a [dot](https://en.wikipedia.org/wiki/DOT_(graph_description_language)) format emitter (ok, I admit it, I probably did this first as I felt it was more fun :P). Just a brief word about dot, dot is a graph description language which can be used an input to [graphviz](http://www.graphviz.org/) to obtain visual representations of a graph. The current implementation is just a minimal dot format emitter and all the features of dot format are not fully supported. At this point, it still cannot be used on any generic graphs. I plan to expand this in the future to allow more features and work on generic graphs.
* The last part was making the CFG out of the emitted RadecoIR. This turned out to be pretty simple, however, there are still a ton of improvements to be made here :)

Apart from Radeco itself, I also helped in making [r2pipe.rs](https://crates.io/crates/r2pipe) which allows communication with radare2 over pipes. In the future, r2pipe.rs will be used to connect Radeco to radare2. R2Pipe.rs is great news if you're a rust person as you can now interact with radare2 and extend it to meet your needs! 

Check out the [documentation](http://radare.github.io/r2pipe.rs/) if you're interested :)

This is just a glimpse of what's in for the upcoming weeks:

* Integration with radare2.
* Write tests and documentation for all features. The limited documentation we currently have can be found [here](http://radare.github.io/radeco). This will be updated soon.
* Convert the IR into the [SSA](https://en.wikipedia.org/wiki/Static_single_assignment_form) form.
* Write [dataflow analysis]( https://en.wikipedia.org/wiki/Data-flow_analysis ) on the SSA form and also perform optimizations like:
  * [Dead code elimination]( https://en.wikipedia.org/wiki/Dead_code_elimination ).
  * [Constant folding](https://en.wikipedia.org/wiki/Constant_folding).
  * [Type propagation]( https://en.wikipedia.org/?title=Constant_folding ).

### Bonus:
Here is a small example of the CFG graph that we're currently about to generate

![cfg](https://raw.githubusercontent.com/sushant94/sushant94.github.io/master/blog/images/cfg.png)