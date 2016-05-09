+++
date = "2016-5-9"
draft = false
title = "Improving analysis"
slug = "Improving-analysis"
aliases = [
	"Improving-analysis"
]
+++

One of the main tasks of Radare2 is to do statically analyse executables. This include binary files disassembly, analysing functions setting [calling conventions](https://en.wikipedia.org/wiki/Calling_convention), auto detecting arguments and type propagation. Autodetecting arguments and type propagation are part of my [Google Summer of Code](https://summerofcode.withgoogle.com/projects/#4786903815553024) task.


New analysis round is added for argument detection. It is architecture independent and supposed to capture all arguments and variables then auto rename them. This analysis round is built on top of [ESIL](https://github.com/radare/radare2book/blob/master/esil.md). It will detect all the `base pointer + num` and store them as arguments and `base pointer - num` and store them as variable. The `stack pointer + num` will always be stored as argument whether it is argument or variable. Identifying the final state of is that `stack pointer offset` is argument or variable is still work in progress. The analysis at the left is the one generated using the new `aa` command, while the one on right is an old instance of the same `aa`.

![analysis](/images/Screenshot at 2016-05-09 22:32:52.png)

`Radare2` also supports renaming declared variable/arguments this can be done using the command `afXn`, where `X` can be:

 - `a` incase of normal arguments
 - `A` incase of [fastcall](https://msdn.microsoft.com/en-us/library/6xa169sk.aspx) 
 - `e` incase of stack pointer is involved
 - 'v' if it is a variable

For example `afan arg_5h my_first_argument` will rename `arg_5h` to be `my_first argument`, You can also set the variable/argument type using the `afXt` where X is the same as that used for `afXn`.

The most important things to know is how to use this analysis round, fortunately it is embedded in the `aa` command, so for general purpose uses you wont need to do anything extra, But there will be a scenario where you define new function at some place where no function existed before. In that case you can enforce this analysis round for the newly created function using `afCa`. It will analyze function located at the current offset and set variables /arguments accordingly.

This is little Example on how to use the new set of commands ;).

[![asciicast](https://asciinema.org/a/74hdvfy6ki59hgis9lv88hozb.png)](https://asciinema.org/a/74hdvfy6ki59hgis9lv88hozb)
