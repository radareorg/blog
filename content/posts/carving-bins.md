+++
date = "2014-06-20T17:36:23+02:00"
draft = false
title = "Carving bins"
slug = "carving-bins"
aliases = [
	"carving-bins"
]
+++
Radare was initially developed as a forensic tool. Nowadays most people use it for static code analysis or binary patching, but the framework and the tools still provide functionalities for analyzing disk partitions or filesystems..

In this post I'm going to explain how to use r2 to extract some ELFs files from a raw memory dump or unknown format firmware image.

This kind of search is called 'carving' and there are already several tools that can do this automatically for free.

* [photorec]( http://www.cgsecurity.org/wiki/PhotoRec )
* [scalpel]( https://github.com/sleuthkit/scalpel )
* [binwalk]( http://binwalk.org/ )
* ...

But this is not the place to talk about them. Let's open an r2 shell!

Searching the needle
---------------

	$ r2 -n dump.bin

At this point we will get the r2 prompt opening the file without trying to understand its format (-n) at the offset 0. We want to search for ELF files, and the '/' command can do this search and store the results in flags.

	[0x00000000]> / \x7fELF
    Searching 4 bytes from 0x00000000 to 0x0000002d: 7f 45 4c 46
	0x00001340 hit0_0
    0x00001744 hit0_1
    ...

On every match we will get a flag pointing to the begining of the ELF files. We can then dump them all to disk with that oneliner:

	[0x00000000]> b 1M
    [0x00000000]> wt @@ hit0*

Note that 'wt' will require an argument (filename) if you are not using the latest version from git. This simplifies the process of naming every dumped portion.

We set the blocksize to 1MB because we don't expect to have bigger executables, but you may change this depending on your target.

Once we have dumped every ELF hit we may like to run rabin2 on them to identify the real size of the original executable by analyzing the file heaers. You can do this with `rabin2 -Z`

	$ for a in dump.* ; do
    sz=`rabin2 -Z $a`     # get RBin.filesize
    r2 -wnqc"r $sz" $a  # resize file
    done

This shellscript loop will truncate all the dumped files to the file size reported by RBin.

Looking for some magic
----------------
Another way for carving for known filetypes on memory is by using the `libmagic` functionality, while we ship a slightly modified code from OpenBSD.

The magic database has been shrinked to just contain the most common filetypes (to speedup search), but you may write or change that magic database to do your own searches and display some header information on every hit.

	/m [magicfile]

The Yara plugin is also distributed with r2. It can be used as a replacement for libmagic and it's already being used for detecting file signatures to determine compiler types, shellcodes, protections and more.

Install latest yara and run `:yara scan` in your r2 shell.


Final words
--------
As long as the IO layer is pluggable, r2 is able to perform those search&dump operations on process memory when attaching the debugger, raw hard disk image, remote files (rap, gdb..), and much more. 