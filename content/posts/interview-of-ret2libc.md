+++
date = "2015-07-16T11:53:57+02:00"
draft = false
title = "Interview of ret2libc"
slug = "interview-of-ret2libc"
aliases = [
	"interview-of-ret2libc"
]
+++
Almost one month since our last article, time flees. This article is an interview of a *new* contributor, that greatly enhanced one of the most visually impressive feature of radare2, the one that our <s>propaganda department</s> contributors loves to show at conferences!

- Who are you ?

> Hi, I'm [ret2libc]( https://github.com/ret2libc ), I was an IDA addicted and this is my 10<sup>th</sup> day that I don't use IDA.

- Hi ret2libc

> Just joking, I still use IDA, but I'd really love to switch in the future, when r2 will be good enough. I am a Computer Science student, I like security related stuff, RE, malwares and... CTFs!

- Why contributing to radare2 ? 

> Just use IDA instead, like everyone else.

> Well, there are just plenty of bugs to fix and it's really fun. r2 is not perfect (yet), but it's an opportunity to be involved in a very good project and full of <s>anal</s> nice people. Also, I was really annoyed by the fact that I have to switch to Windows or a Windows VM everytime I just want to analyze a binary, even a simple one.

> Instead r2 works almost everywhere, I just need my terminal. It's free, it's cool and it can be a good enough tool to work with binaries from some CTF. 

> So, let's try it, but... wait, there's only plain disassembly here, where is my graph?? Ok, `VV` and, well, I've found what I wanted to improve ;)

- You have fixed a big pile of bugs regarding graphs, that's pretty impressive. Do you have a strong background in programming/maths ?

> Let's say I have a background in programming/maths. As I mentioned, I'm a Computer Science student.

- Why the graph?

> I think it's one of the fundamental feature a disassembler should have. It makes you understand at a glance what's going on in a program and many times it's the only thing I need from IDA.

- What are you currently implementing ? [Someone](https://twitter.com/r2gif/status/621291945365667840 ) told me about colours in graphs.

> I've just finished implementing an algorithm to have a better layout for the nodes of the ASCII-Art graph, so that you won't have anymore (at least, you shouldn't) overlapping nodes or nodes placed in really wrong positions. Don't expect IDA's graph, but it's already something ;)

>  Since, at the moment, you can only see one function in the ASCII-Art graph, currently I'm implementing a way to move between called functions and go back and forth between them, without having to leave the graph mode. Something like what you get in Visual Mode with `[0-9]` shortcuts ([#2907]( https://github.com/radare/radare2/issues/2907 )).

> For the colours in graph, I think you will have to wait a little bit. In the meantime, if you feel brave you can start to use 'VV!', but don't say I didn't warn you.

- What is the plan for the next months ? Are you going to continue to work on r2 ?

> Of course I will continue contributing to r2, in my spare time! I'm planning to focus on graph, as I've done lately. In particular I think you will see:

> * [issue #2907]( https://github.com/radare/radare2/issues/2907 ) fixed: move between called functions with [0-9] keys
> * enhancement in the edges layout. They are a real mess at the moment and for a good enough graph it's one of the fundamental thing that has to be fixed.
> * mouse support for node selection
> * last, and also least *:P*, you will see colored disassembly in the graph, unless someone else want to implement it before me (you are welcome!!)
> * a lot of other cool stuff, but that's another story and everything else really depends on having a good and strong starting point.

- What about, a GUI ?

> What? GUI? Pff, we just have the terminal :) 

> Really, I'd really like to see a cool GUI for r2, but don't count on me.

- Most hated/lover feature of r2 ?

> * Most hated feature: `anal.hasnext` set to true as default!
> * Most loved feature: ASCII-Art graph, of course ;)

- Advices for new contributors and users ?

> Contributors, just focus on something you'd like to be present in r2 and implement it, maintain it and add tests!!

> Users, keep using IDA, unless you want to take the red pill and see how deep the rabbit-hole goes ;)

- Last word ?

> Maybe we will see r2 1.0 in the future! VVRRRRRRRRRRRRRRRRRRR

- Screenshots ?


![Graph example](/blog/images/full.png)
*Example of graph*

![Disasm in graph](/blog/images/disasm.png)
*Disassembly in graph, without colours (yet)*
