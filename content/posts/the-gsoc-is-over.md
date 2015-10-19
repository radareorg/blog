+++
date = "2015-08-25T13:13:29+02:00"
draft = false
title = "The GSoC is Over!"
slug = "the-gsoc-is-over"
aliases = [
	"the-gsoc-is-over"
]
+++
Good news everyone! Our first time participation in the [Google Summer of Code](https://developers.google.com/open-source/gsoc/), thanks to our previous and current experience of the hosting of the Radare Summer of Code, was a **great success**. It wouldn't have been possible without the help of the great [Solar Designer]( https://en.wikipedia.org/wiki/Solar_Designer ), who took us under [Openwall]( http://openwall.com/ )'s project umbrella for the GSoC.

We had two GSoC students successfully complete tasks related to [Radeco](https://github.com/radare/radeco), our new, work in progress, **decompiler**.

Our students have done an amazing job, learning radare, learning [Rust]( https://www.rust-lang.org/ ), and becoming important parts of the radare community.

At first, we should note that they have actively participated in the deep research of the existing decompilers, papers about [SSA]( https://en.wikipedia.org/wiki/Static_single_assignment_form ) and decompilation process and even some books. This allowed them to dig into this area of knowledge and design a brand new graph-based intermediated representation, which took all the best from the various practical and academical ILs/IRs.

At second, they were able to learn new language Rust quite fast, and unleash the whole power of the types checking to make their life easier and (sometimes) improve the types system/algorithms itself.

By the end of the summer, they have written a combined total of almost 7,000 lines of rust to create utilities for converting ESIL to a new SSA IL called RadecoIR, complete with constant propagation, dead code elimination, and soon, some great pseudo-c output!

We'd like to thank Google's Open Source Programs for hosting the GSoC, Openwall (especially solardiz) for accepting us as a subproject, mentors xvilka and crowell, pancake for constantly providing support, and of course our students [sushant94]( http://sushant94.me/ ) and [dkreuter]( https://github.com/dkreuter ) for making this possible.

If you have the chance, give [radeco]( https://github.com/radare/radeco ) a try, we'd love to see your bug reports, suggestions, and especially your pull requests ;D