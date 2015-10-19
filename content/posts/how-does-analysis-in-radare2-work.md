+++
date = "1970-01-01T01:00:00+01:00"
draft = true
title = "How does analysis in radare2 work?"
slug = "how-does-analysis-in-radare2-work"
aliases = [
	"how-does-analysis-in-radare2-work"
]
+++
Code analysis is an important tool for any reverse engineer, doing proper automatic processes to understand how the code is structurated and what it does helps the disassembler to produce more readable disassembly.

Unfortunely, that process is not an low hanging fruit. And it's even more hard if you plan to do it in a generic way across many different architectures.

Radare2 was born with portability in mind. And this is why historically have been shipping a single code analysis engine.

After so much more testing and feedback from users we have noticed bugs, regressions and random problems that can make a program be analized wrongly.

To improve the situation I have been working on extending the code analysis to handle some config variables, use an alternative code analysis engine and even wrote a freshly new plugin to perform that using Sdb primitives.

The good of all this history, is that r2 provides a very low level tools to manipulate and perform handmade analysis with just a single line:

```
> s entry0
> asm.filter=false
> af @@= `pi $SS@$S~call[1]`
```

This command will disassemble the whole current section (.text), grep for 'calls' and retrieve the destination offset of each result.

Then it runs 'af' (analyze function) on each address.

As long as this trick is pretty useful and common, I have added a command `ac` to do it automatically.

First run
-------
Most users will run `r2 -A`or the `aa` command in the r2 prompt just after opening a program. This command stands for _analyze all_ and what it does internally is `af @@ sym*`. Which runs af command on every symbol.

The `af` command analyzes a function starting at the current seek. In case `anal.hasnext` has been set it will try to find a function right after the end of it. And will recursively analyze every function inside a CALL.

Options
-----
As you have seen, the analysis can be tuned to perform better on some situations. 

anal.hasnext
anal.nopskip
anal.depth

Hand made analysis
-------------
af+
afbb+ 
...


Trampolines
----------

Preludes
------
It is also possible to define a function prelude signature (in hexadecimal), to specify where the functions do start (usually, most functions use the common pattern bytes at the begining of the functions.

The `ap` command will do that for you, and in case you want to use a different prelude you can define it in `e anal.prelude`.

```
> ap
```
In r2 scripting this would be:
```
> /x 894548
...
> af @@ hit*
```

Visual mode
---------
A function can be analized in visual mode by pressing the `df` keys. This will define the current offset as a _function_. If you look closely to the 'd' output you'll notice that this menu allows you to perform different actions on the metadata and analysis modules.

Which opens the door to resize function boundaries, change the format type (code, data, ..), rename the function, change the word size, etc..

```
Vdu -> undefine medata
Vdr -> rename function
Vdf -> analyze function
Vdw -> show as 32bit number
VdW -> show as 64bit number
Vdj -> merge down current function with the above
Vdk -> merge up current with previous function
...
```

String references
------------

Alternative 
--------
TODO: a2f core plugin

Future
-----
ESIL (2nd layer analysis)

--pancake