+++
date = "2014-05-19T02:02:01+02:00"
draft = false
title = "Loading iOS binaries"
slug = "loading-ios-binaries"
aliases = [
	"loading-ios-binaries"
]
+++
There are several posts explaining the process to decrypt an iOS app, this is not new, but no one explained the instruction to do it with r2.

We have no aim in promoting piracy or cracking, but that's the only way to analyze applications from the AppleStore.

Retrieving information
----------------

First of all you need a jailbroken device, in [Cydia]( http://cydia.saurik.com/ ) add the `cydia.radare.org` repository, in order to get the latest radare2 package from there. Once installed, ssh into the device and type the following commands to retrieve the encryption information stored in the MACH0 header.

```
$ export APP=/var/mobile/Applications/*/Target.app/Target
$ rabin2 -k 'info/*' $APP
uuid.0=034324c306b633909dbd0f229ceae7c9
crypto=true
cryptid=0x1
cryptoff=0x4000
cryptsize=0x6e4000
```

At the moment, only few information are stored in SDB, but this will change during next months. You may also access this information from the r2 shell this way:

```
$ r2 $APP
[0x00004320]> k bin/cur/info/*
```

Let's create a simple r2 script to add two flags to store *cryptoff* and *cryptsiz*e variables that we may use later for dumping and patching.

```
$ cat > bin.r2 <<EOF
f cryptoff=0x4000
f cryptsize=0x6e4000
EOF
```

Dumping the memory from the debugger can be done with the following commands:

```
$ r2 -i bin.r2 -d $APP
[0x2be78028]> dm
sys 0x00000000 - 0x00001000 s --- --- 00 share/priv/not-reserved
sys 0x0002d000 - 0x00031000 s r-x r-x 01 copy/priv/not-reserved
sys 0x00031000 - 0x00715000 s r-x r-x 02 copy/priv/not-reserved
sys 0x00715000 - 0x00889000 s rw- rw- 03 copy/priv/not-reserved
sys 0x00889000 - 0x0088d000 s rw- rw- 04 copy/priv/not-reserved
sys 0x0088d000 - 0x00909000 s r-- r-- 05 copy/priv/not-reserved
sys 0x27cd7000 - 0x27cd8000 s --- rwx 06 copy/priv/not-reserved
sys 0x27cd8000 - 0x27dd7000 s rw- rwx 07 copy/priv/not-reserved
sys 0x2be77000 * 0x2be98000 s r-x r-x 08 copy/priv/not-reserved
sys 0x2be98000 - 0x2be99000 s rw- rw- 09 copy/priv/not-reserved
sys 0x2be99000 - 0x2bec1000 s rw- rw- 0a copy/priv/not-reserved
sys 0x2bec1000 - 0x2becf000 s r-- r-- 0b copy/priv/not-reserved
sys 0x2c000000 - 0x3c000000 s r-- rwx 0c share/priv/reserved
sys 0x3c000000 - 0x40000000 s r-- rwx 0d share/priv/reserved
[0x2be78028]> s `dm~r-x:0[1]`
[0x00040000]> s+cryptoff
[0x00044000]> wt dump.bin cryptsize
dumped 0x6e4000 bytes
```
Patching the binary
--------------

Once we get the dump we will want to patch the program. We will ignore the *RBin* information (`-n`) and enable the write mode option (`-w`). This will load the file in '*raw*' and we will patch with the decrypted block.

```
$ cp -f $APP bin
$ r2 -i bin.r2 -nw bin
[0x00000000]> wf dump.bin @ cryptoff
```

We will have to patch the binary to mark it as unencrypted. Setting `cryptid` to **0**.
```
[0x00000000]> /v cryptsize
[0x00000000]> s hit0_0+4
[0x00000a08]> wx 00
```

Loading Objective-C metadata
--------------------

Most iOS apps are written in ObjC, and all the symbolic information are stored there, this is very useful to understand the app workflow and design.

At the moment, the way to load ObjC information into radare2 is by using the `objc.pl` script which makes uses of `class-dump`.

```
$ ~/radare2/doc/objc.pl bin > bin.objc.r2

```

Or directly from the r2 shell

```
[0x00004000]> .!./objc.pl $FILE

```

Extracting sub-bins:
--------------
As a final note, just say that you may find iOS apps in MACH0 or FATMACH0 formats, in the last case we will be interested in extracting and analyzing only the one that fits best to our needs.

For example, inside a FATMACH0 iOS app you can find ARMv6, ARMv7 and ARM64 executables. The tool `rabin2` can be used to list all sub-binaries contained inside a FATMACH0 file using the `-A` flag:

```
$ rabin2 -A bin
```

Also we can use the `-x` flag to extract them all.
```
$ rabin2 -x bin
```

Or we may need to specify which sub-bin we want to load by using the `r2 -a <arch> -b <bits>` flags when loading a FATMACH0.
