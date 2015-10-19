+++
date = "2014-04-30T23:30:45+02:00"
draft = false
title = "YARA support"
slug = "yara-support"
aliases = [
	"yara-support"
]
+++
We now have (experimental) [YARA]( https://plusvic.github.io/yara/ ) support inside radare2.

If you are building from the latest git, you just have to install `libyara`, no need to recompile anything.

```
[0x00000000]> yara
Yara plugin
| add [path] : add yara rules
| clear      : clear all rules
| help       : show this help
| list       : list all rules
| scan       : scan the current file
[0x00000000]> 
```

Since you may not already have some rules, we bundled some defaults ones, for packers and [crypto primitives]( https://github.com/Phoul/yara_rules ).