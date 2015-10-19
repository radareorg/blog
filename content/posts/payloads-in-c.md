+++
date = "2014-08-17T16:15:17+02:00"
draft = false
title = "Payloads in C"
slug = "payloads-in-c"
aliases = [
	"payloads-in-c"
]
+++
Writing exploits requires to perform several steps to achieve the final purpose of the attack.

* find a vulerability
* reverse engineer the bug
* achieve code execution
* write the payload
* profit

This post will focus on the later step: write the payload.

The payload can spawn a shell, reuse a socket or do a connect back. But sometimes we will need a more complex payload that will need to open a file, change some permissions, do some mmap, etc.

We can use different tools to do this:

* [metasploit]( http://www.metasploit.com/ ) to cook a generic shellcode and encode it
* [inline-egg]( http://corelabs.coresecurity.com/index.php?module=Wiki&action=view&type=tool&name=InlineEgg )
* [shellforge]( http://www.secdev.org/projects/shellforge/ )

But r2 already provides its own functionalities to do this with ragg2. Ragg2 implements the following features:

* generic shellcodes
* xor encoder
* tiny binary construction via rabin2
* specific low level lang
* C compatible lang to shellcode (based on shellforge)

We will focus on the last option. It's the most flexible, but have a runtime dependency on gcc/clang.

Halp
----

Let's first run `ragg2-cc -h` to see what it can do for us:

    $ ragg2-cc -h
    Usage: ragg2-cc [-dabkoscxv] [file.c]
      -d       enable debug mode
      -a x86   set arch (x86, arm)
      -b 32    bits (32, 64)
      -k linux set kernel (darwin, linux)
      -o file  set output file
      -s       generate assembly
      -c       generate compiled shellcode
      -x       show hexpair bytes
      -v       show version

Compiling a shellcode
---------------------
At this point we are probably not interested on memory constraints, but we still need a simple way to express code.

Shellforge was abandoned a while ago, and it was done in Python. Ragg2 just replicates the same functionalities in few shellscript lines and aims to keep the include directory updated.

We just need to write C code, using syscalls, and avoiding libc or other library calls. For example:

    $ cat hi.c
    int main() {
      write (1,"Hello!\n",7);
      exit(0);
    }

We can now compile this with ragg2-cc:

    $ ragg2-cc -x hi.c
    eb00488d351d000000bf01000000ba07000000b8010000000f0531ffb83c0000000f0531c0c348656c6c6f210a00

The -x flag will dump the shellcode in hexadecimal to stdout.

We can then disassemble this using rasm2 -d.

Under the hood
--------------
What ragg2-cc is doing is basically replacing the -isystem include path and compile the program in relocatable mode (-fPIE).

Then it will just dump the whole text section, prepending it with a single jmp to main.

We can feed that hexpair thing to ragg2 or metasploit to encrypt or encode it before sending the payload to the target.

--pancake