+++
date = "2014-11-17T08:20:09+01:00"
draft = false
title = "The RSoC is over"
slug = "the-rsoc-is-over"
aliases = [
	"the-rsoc-is-over"
]
+++
October is over and we delayed a bit the end of the [RSoC]( http://rada.re/rsoc/ ) in order to get everything done for the [release]( http://radare.today/radare-0-9-8/ ), and it seems that little happened as planned:

The RSoC advertisement was a great opportunity to get new developers interested in contributing to the project, some of them even without joining the RSoC took some points that weren't requested and delivered them! That's pretty cool, because our two *selected students* disapeared during the summer.

## Abandonned tasks
We're a bit sad that the [sdbization]( http://radare.today/exploring-the-database/ ) task wasn't completed, since this would have been a huge improvement, both in term of performances and cool new features, like collaboration. Also, no one gave some love to the [web interface]( http://cloud.rada.re ), so it will still be an ugly PoC a bit longer.

## Complete tasks
Most of the tasks weren't completed on the test cases and documentation sides, but we hope to get things done in time for the [next release]( https://github.com/radare/radare2/milestones/0.9.9 ), the 0.9.9.

We'd like to congratulate our 3 volunteers for their hard work and dedication. They were supposed to be the 'free' team, but since our two *official* students left, we'd like to pay them as if they were selected: We managed to get **$2100** in budget, so each of them will receive **$700**.

### Structures support
[Skia]( http://libskia.so/ ) did great to implement structures support, a bit *à la* 010Editor.
![structure](/blog/images/687474703a2f2f7777772e6c6962736b69612e736f2f7075622f72322532306c696e6b65642532306c6973742e706e67.png)
Now with `r2 -nn` it is possible to analyze the file header structs using the `pf.`, `pxa` and other related commands.

```
$ r2 -nn /path/to/bin
```

### FLIRT and YARA support
[jfrankowski]( http://failhard.org/ ) did a great RSoC, and improved [yara]( https://plusvic.github.io/yara/ ) support in radare2, and also added the [FLIRT]( https://www.hex-rays.com/products/ida/tech/flirt/in_depth.shtml ) one! Currently, radare2 is only able to use existing signatures, but feel free to drop us a patch to build our own, using radare!

Yara3 support is almost there, but we prefer to release for a welltested yara2 version and push the upgrade in 0.9.9.

### PDB support
[inisider]( https://github.com/inisider ) implemented a standalone library to handle [PDB]( https://support.microsoft.com/kb/121366 ) files, and integrated it into radare2. You can now analyse/debug PE with much more ease.

```
> .!rabin2 -rP test.pdb
```

### ESIL
Nobody took that task from the RSoC, but pancake and condret raised the bar and managed to get a working implementation of [ESIL]( https://github.com/radare/radare2/wiki/ESIL ), mainly tested on gameboy, brainfuck, x86 and mips. Also, pancake made a new search command to use esil expressions to perform complex conditional carving useful for forensics and data analysis.

## Conclusion
We enjoyed holding this first RSoC. We made some mikstakes, and learned a lot of things. If we won't get selected for a GSoC once again, be sure that the second RSoC will be better than the first one.

We'd like to thank everyone who attended, participated, donated, supported, helped, advertised, improved and tested things. Without you, the RSoC wouldn't have been so productive!