+++
date = "2014-09-18T14:17:12+02:00"
draft = false
title = "Scripting r2 in Vala"
slug = "scripting-r2-in-vala"
aliases = [
	"scripting-r2-in-vala"
]
+++
Under some situations you need to automatize or extend the features of radare. There are so many scripting languages out there: python, ruby, perl, lua between others.

All of them are supported by the radare package and you can use them from inside r2 using r_lang plugins and the '#!' command or externally with the r2-swig.

The main issue on scripting languages is performance. The code is interpreted and all the api bindings are wrapped, so linked list accesses and function calls are highly penalized.

Here's where Vala joins the party.

Vala compiles into C and generates native code with no wrappers, but providing a higher-level interface than C, so it is harder to segfault.

Let's see how to run scripts for r_lang in the r2 prompt:

```
[0x8048520]> #!
 vala: VALA language extension
```

The '#!' (hashbang) command is used to invoke the r_lang REPL prompt or run the code of the given file.

The command has reported that it our build of r2 has built-in support for running Vala code from the core. So let's try it using the read-eval-print-loop mode.

```
[0x8048520]> #!vala
vala> print ("%p\n", core)
0x804b2c0
```

This is what it's happening after each newline in the prompt:

* wraps line with: 'using Radare; public static void entry(RCore core) {}
* compiles temporary .vala file into a shared library against r_core library
* loads the library and resolves the symbol named 'entry'
* calls the entry point with the RCore instance as argument
* unloads the library and removes temporary files 

You can also write the vala code in a separated file named 'foo.vala':

```
[0x80498d2]> cat foo.vala 
using Radare;

public static void entry(RCore core) {
        core.cmd0 ("pd 2");
        core.cons.flush ();
}
```


Execute this file with '#!vala foo[.vala]':

```
[0x80498d2]> #!vala foo
  0x080498d2        5e  pop esi
  0x080498d3      89e1  mov ecx, esp
```

For more documentation on bindings see the [vapi](http://radare.org/vdoc) documentation.