+++
date = "2014-05-10T20:40:10+02:00"
draft = false
title = "Mitigations detection"
slug = "mitigations-detection"
aliases = [
	"mitigations-detection"
]
+++
Since the [Smashing The Stack For Fun And Profit]( http://insecure.org/stf/smashstack.html ) article from Aleph1, a lot has been done on mitigation side: [canaries]( https://en.wikipedia.org/wiki/Stack-smashing_protection#Canaries ), [DEP]( https://en.wikipedia.org/wiki/Data_Execution_Prevention )/[W^X]( https://en.wikipedia.org/wiki/NX_bit#Software_emulation_of_feature ), [PIC]( https://en.wikipedia.org/wiki/Position-independent_code ) (to allow [ASLR]( https://en.wikipedia.org/wiki/Address_space_layout_randomization )), [RELRO]( http://tk-blog.blogspot.fr/2009/02/relro-not-so-well-known-memory.html ), [SafeSEH]( http://msdn.microsoft.com/en-us/library/vstudio/9a89h429%28v=vs.100%29.aspx ), ...

Because radare2 is also designed to be a present in the exploit writer arsenal, *jvoisin* implemented detection for some of those mitigations.

## GNU/Linux
- GCC's canary implementation can be detected by the presence of the [__stack_chk_fail]( http://refspecs.linux-foundation.org/LSB_4.0.0/LSB-Core-generic/LSB-Core-generic/libc---stack-chk-fail-1.html ) function. It is used to terminate a function, in case of stack overflow.
- *PIE* can be detected by the presence of sections headers of type *dynamic*.

Unfortunately, radare2 is not able to detect RELRO and NX for now.

## Windows
- On windows, the *DEP* and *PIE* features can be detected by [parsing the header]( http://0xdabbad00.com/2013/09/02/file-scanner-web-app-part-3-of-5-yara-signatures/ ).
- Visual Studio's canary implementation can be detected by the presence of a *__security_init_cookie* function. 

Radare2 is not yet able to detect SafeSEH.