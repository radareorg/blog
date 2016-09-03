+++
date = "2016-08-25T13:13:29+02:00"
draft = false
title = "GSOC, The last commit 213c6f"
slug = "GSOC-The-last-commit-213c6f"
aliases = [
	"GSOC-The-last-commit-213c6f"
]
+++
It has been so long since I posted, the reason is that I have been quite busy with my [GSOC](https://summerofcode.withgoogle.com/projects/#4786903815553024) timeline. Yes, I know I said I will write blogs once every week, but I learned the hard way how not to do such promises again ;).

I guess it is time to summarize the whole 3 months work, but before I start, I would like to state that all my contributions are in basically 2 repositories: [radare2](https://github.com/radare/radare2) and [radare2-regression](https://github.com/radare/radare2-regressions), you can find my contributions [here](https://github.com/radare/radare2/commits?author=oddcoder) and [here](https://github.com/radare/radare2-regressions/commits?author=oddcoder) for both repositories.

So what I have done during the past three and a half months?

## Fixing `t` commands
These commands had lots of regressions inside, it is basically using interpreter based on [tcc](http://bellard.org/tcc/) to convert C into `key=value` pairs, my work here was mainly fixing bugs related to querying and editing values in the key/value form. I didn't touch tcc code because it is big system on its own.

## Storing formal parameters and local vars
Before I started, r2 was only able to store local vars that are offseted from `ebp`, actually since ebp only existed in `x86` architecture, the approach never worked on other architectures like mips or arm. I added the support of 3 different local vars (locas) types  and formal arguments (args):
- Base pointer based args/locals.
- Stack pointer based args/locals.
- Register based args.

Those are integrated completely into r2 and replaced the old implementation. They replace their offsets in disassembly, one also have the option to display their value while debugging or emulating, set their types etc... One limitation is that it will never work for custom stack configuration for example when using `eax` as base pointer and say `ecx` as stack pointer.

## Auto detecting formal parameters and local vars
Everything I implemented here have something to do with [ESIL](https://github.com/radare/radare2book/blob/master/esil.md) one way or another. Actually ESIL is always meant to be emulated, by the time I was implementing this I've never understood how emulation work, so I figured another hackish way that later on found to be much effective than emulation. I just did some kind of text matching on esil!, all additions or subtractions from base pointer, stack pointer will be noted argument or var, that is it. no memory tracing (as we will see in type matching) no check for traps or zero dereferencing, also no worries about code coverage.

One of the limitations of this approach is that all detected locals /arguments will be of word size, don't confuse word here with Microsoft's own WORD, I mean the maximum amount of data a CPU can process at a time, so variables that are say struct will never be detected as a struct at this phase of analysis.

## Calling convention profiles
The idea behind [calling convention](https://en.wikipedia.org/wiki/Calling_convention) is that every architecture, or probably every compiler put set of rules that regulates the relation between the caller and callee functions. At first, all calling conventions were part of the source code, hardcoded in C, and doesn't include any helper APIs. Actually all of the available calling conventions were loaded for all architectures, the default one was [cdecl](https://msdn.microsoft.com/en-us/library/zkwh89ks.aspx), based on that some interesting behavior occurs like for example you may see a arm function that is claimed to be called by cdecl. Luckily enough nothing critical in the code base depended on calling conventions at that time.

What I tried to achieve is to isolate code from data, make people able to contribute to calling conventions as data that doesn't require knowledge of r2 code base, load calling conventions for each architecture on its own so no cdecl on mips arch, etc. I have full description of how to create calling conventions profile for your arch and how to integrate them into r2 so they load automatically [here](https://github.com/radare/radare2/blob/0.10.5/doc/calling-conventions.md).

## Type Matching
To be honest, this is considered the most challenging part for me in the whole GSOC experience, far until that moment I had no idea how does esil works, (remember, I always avoided using it for emulation). I needed some advanced features in esil that I was never sure whether it existed in first place or not, such as viewing memory register accesses, undo last (N) step into commands. I also had a wrong understanding of emulation, that it will always be as perfect as a debugger, and I learned that it is not the hard way. Actually ESIL is very very accurate on its own, but before the target function is emulated, program loader does lots of magic that ESIL never do. Emulating function calls was the worst part actually instruction pointer always ended to be `0x0` which is really really bad. Luckily with huge amount of help from my mentors all those issues were solved, I was so lucky that at least one other student in [RSOC](http://radare.org/r/rsoc.html)(radare2 own version of GSOC) had the same problem, and we cooperated on figuring out what to do. And it is solved to some extent.

At this point what I was going to do is to merge all what I did before (all together) + help of esil to obtain type information about arguments passed to each standard function. Function names, arguments and calling conventions are taken from types databases, description of the target calling convention is taken from calling convention databases, memory/registers accesses are taken from ESIL emulation of the function, all together will help identifying which instruction is passing argument to the callee function. More over if that instruction holds an arg/local for the caller it will be given the same type of the that argument of the callee.

Everything sounds sweet, except for the fact that emulation will always follow always one path, so full code coverage wont be achieved in case if their is conditions or loops in functions which (almost every function do have), This is the basic limitation of that approach to match types for standard functions.

## What is next
I got lots of plans for what to do next since GSoC is over. I loved r2 as both developer and user so I will stick to it. Plus it is one of my priorities to make that project suit my needs. At least because it wont be very convenient to buy something like the $4.7K IDA Pro just to play around with it. Most probably I will apply for next year GSoC as mentor for radare2, assuming I will be ready for that by 2017 spring.

For sure what I will be working on for now is documenting types, since `tcc` needs lots of work till it generate the desired sdb key/value pairs, people need to know how to use create those databases manually without grepping through code base. Since I cant create all the databases my self I need help from others so this will be priority one.

Also I've started to work on some of the Equation group leaked malware, which is arrived in time. It will be a good exercise for me and wont disturb me from contributing to radare2, because they will expose a treasure trove of bugs everywhere (I already found one).

As I said before, type matching lacks 100% code coverage, so I need to solve this problem as well. The apparent solution will be to use symbolic execution engine. I learned that [radeco-lib](https://github.com/radare/radeco-lib) have that capabilities (or can be easily implemented there). That will require learning rust and getting familiar with radeco-lib codebase. But that certainly worth the effort!

Building metamorphic engine for radare2. I don't even know in details what that is but it got some nice article on Wikipedia and new python repository that was hosted on github, plus it help me to understand how to work with malware that deploy that kind of technique deeply.

Contribute to radeco: one of my mentors recommended that to me. It sounds like a great idea, since radeco needs a lots of work plus I enjoy doing new things regularly.

Helping ESIL to emulate loaders: for this I will need to be file format ninja to understand how loaders work (most likely I will work on elf files).

Adding support for TIL files: those files are used by IDA to store data types information for major compiler and APIs this will be way helpful for IDA users who are moving to r2.

## For next year students
I have learned a lot from previous students/mentors hints. And as a GSoC participant duty, I should give next generation of students some kind of advice, so they never get surprised. That feedback wouldn't differ a lot from what others would tell, write code a lot, keep strong bonds with your mentor, etc... But their is one thing that no one ever mentioned and I learned it by doing.

Although Google allows you to apply for more than one organization, it is much more useful to just apply for one and focus on contributing to that one as much as you can. Applying for more than one organization will be sort of distraction in my opinion. If this is your first time in open source projects, things will look too complex for a while. Actually it will be long period where you are just astonished how things got done. If you have that feeling probably you are on the right track. All what you need is some self confidence, patience, read as much of the newsletters or IRC logs or whatever available, ask questions, don't get embarrassed by asking stupid question - after all you will get the answer.

One last advice, stick to your project as long as you can after GSoC, probably this is the one main target of GSOC, to get students introduced to open source projects. If you haven't done that - shame on you :P.
