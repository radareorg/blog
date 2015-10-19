+++
date = "2014-09-16T15:40:28+02:00"
draft = false
title = "Solving crackmes with LDPRELOAD"
slug = "solving-crackmes-with-ldpreload"
aliases = [
	"solving-crackmes-with-ldpreload"
]
+++
This is a translation of [this]( http://www.securitybydefault.com/2014/09/solucionando-crackmes-precargando.html ) article.


One of the most common technics used in UNIX for analyzing and modifying a program consists in preloading a library to make the dynamic linker priorize the functions in there before the ones coming from external libraries.

In fact, in iOS, the whole [MobileSubstrate]( http://iphonedevwiki.net/index.php/MobileSubstrate ) thing and the [Flex app]( https://www.adobe.com/products/flex.html ) are based on this concept to extend and modify the functionalities of the applications in a very simple way.

The procedure requires defining an environment variable (`LD_PRELOAD` or `DYLD_INSERT_LIBRARIES` in iOS/OSX). It is then parsed by the [dynamic linker]( https://en.wikipedia.org/wiki/Dynamic_linker ) (ld.so or dyld) which will load the library allowing us to: 

* inspect function parameters and contents
* change the return value of a function
* extend its functionalities
* replace the implementation
* dump backtraces
* ...

To show up this theory I wrote a small crackme:

     I2luY2x1ZGUgPHN0ZGlvLmg+CiNpbmNsdWRlIDxzdHJpbmcuaD4KCgppbnQgbWFpbihpbnQgYXJnYywgY2hhciAqKmFyZ3YpIHsKCWNoYXIgKnBhc3MgPSAoY2hhciAqKWFyZ3ZbMF07CglpZiAoYXJnYzwyKSB7CgkJcHJpbnRmICgiR2ltbWUgYW4gYXJnXG4iKTsKCQlyZXR1cm4gMTsKCX0KCWlmICghc3RyY21wICgiYSIsICJiIikpIHsKCQlwcmludGYgKCJBcmUgeW91IHRyaWNraW5nIG1lP1xuIik7CgkJcmV0dXJuIDE7Cgl9CglpZiAoIXN0cmNtcCAoYXJndlsxXSwgcGFzcykpIHsKCQlwcmludGYgKCJZb3Ugd2luIVxuIik7CgkJcmV0dXJuIDA7Cgl9CglwcmludGYgKCJXcm9uZ1xuIik7CglyZXR1cm4gMTsKfQo=

The disasm of that crackme in OSX looks like this:

![](http://radare.org/img/post/dis-text.png)

Injecting libraries is only possible on non-static, non-suided executables. This can be checked with `ls` and `rabin2`:

	$ isSuid() { ls -l $1 |awk '{print $1}' | grep -q s && echo YES || echo NO; }
	$ isStatic() { rabin2 -I $1 | grep ^static | grep -q true && echo YES || echo NO; }

	$ isSuid ./a.out
	NO

	$ isStatic ./a.out
	NO

We're lucky! This crackme that executable is fine to continue our explanations ;)

At this point we must know which symbols and libraries do the program import or use. For that we will use `rabin2`, a program that comes with `radare2` and will allow us to inspect and extract information from binaries, mostly executable programs and libraries.

	$ rabin2 -qi a.out
	printf
	strcmp
	dyld_stub_binder

	$ rabin2 -ql a.out
	/usr/lib/libSystem.B.dylib

Fine, looks like the executable is using `printf` and `strcmp`. We can asume that the program is using strcmp to verify the password provided by the user. So we will write a small library that will override the return value of it whatever which parameters it takes.

	$ cat mylib.c
	int strcmp(char *a, char *b) { return 0; }
	$ gcc -fPIC -shared mylib.c -o mylib.dylib

	$ rarun2 program=./a.out preload=mylib.dylib arg1=something
	Are you tricking me?

If we change the return value, the crackme will detect it, because at first is doing an always-false conditional that will spot the hooker.

So, we need to create a condition or filter to make it return 0 only when called from the password check 'call'.

In this example we decided to reimplement `strcmp` to avoid making the example more complex and avoid explaining/depending on lazy binding and rtld-next features.

I wrote this (arch/compiler-specific) macro to retrieve the address of the caller.

```
#define GETCALLER(x,y)\
	void *x = (void*)*(&x+2+y);

int strcmp (char *a, char *b) {
	GETCALLER(caller, 2+1); // 2 args + 1 import redirect
	if (caller == 0x100000ee7) {
		write (1, "INT\n", 4);
		return 0;
	}
	int al = strlen (a);
	if (al != strlen (b))
    	return 1;
	return memcmp (a, b, al);
}
```

The magic number 0x100000ee7 is the address of the next instruction after the `call strcmp` of the caller that we want to intercept, we takes this address because is the one stored in the stack.

![](http://radare.org/img/post/strcmp.png)

We compile the library and execute the crackme with rarun2 to make him define the environment and inject the library in the program via LD\_PRELOAD method or DYLD\_LIBRARY\_INSERT depending if running on Linux/BSD or OSX.

	$ gcc -shared -fPIC mylib.c -o mylib.dylib
	$ rarun2 program=./a.out preload=mylib.dylib arg1=something
	You Win!

It's important to say that preloading a library allow us to introduce hooks on the calls to external libraries. This is.. we can't hook internal program calls.. or we do?

In the case of having the possibility to modify the program we can patch the calls to the function we want to hook to an external simbol in the PLT, and then let our LD_PRELOAD implementation decide which caller is accessing the hook and perform a redirect to the original destination.

This way we can hook internal or external function calls without having to reallocate the executable program.

Let's make an example to make it clear:

	$ cat test.c
	int test(char *a) {
		printf ("Testing arguments.. ");
		if (!strcmp (a, "foo")) {
			printf ("LE WIN!\n");
			return 0;
		}
		printf ("FAILE!\n");
		return 1;
	}
	int main(int argc, char **argv) {
		return test (argv[0]);
	}

We build and disassemble the program

	$ gcc test.c
	$ r2 -Aqc 'pD $SS @ $S' a.out

![](http://radare.org/img/post/dis-text2.png)

As we can see, the `sym._text` is defined at `0x100000e90`. If we want to reimplement that function with an external one we will need to find a target import and write our call proxy in there, deciding which action take depending on the address of the caller. Import symbols are placed in the process memory and are overwriten by the dynamic linker to set the address of the mapped address of the library in memory.

	$ rabin2 -i a.out
	[Imports]
	ordinal=000 plt=0x100000f38 bind=NONE type=FUNC name=printf
	ordinal=001 plt=0x100000f3e bind=NONE type=FUNC name=strcmp
	ordinal=002 plt=0x000000000 bind=NONE type=FUNC name=dyld_stub_binder

	3 imports

	$ rabin2 -s a.out | grep imp.
	vaddr=0x100000f38 paddr=0x00000f38 ord=003 fwd=NONE sz=0 bind=LOCAL type=FUNC name=imp.printf
	vaddr=0x100000f3e paddr=0x00000f3e ord=004 fwd=NONE sz=0 bind=LOCAL type=FUNC name=imp.strcmp

Once we get the address of the imports we can proceed to patch the program:

	$ r2 -qc 'wa jmp 0x100000f38 @ 0x100000e90' -w a.out

If we disassemble `sym._test` now, we can observe a call to `sym.imp.printf`:

![patched code](http://radare.org/img/post/dis-pd.png)

If we run the program now we will be able to see the expected password, because we are displaying the argument passed to `strcmp`

	$ ./a.out
	./a.out

And then we run the original crackme with the correct password to verify:

	$ ./a.out.orig ./a.out.orig
	LE WIN!

For this example we didnt felt the need to investigate much further, but this PoC clearly shows how to use r2 to inject *LDPRELOADed* libraries and how to patch executables. And mixing this with a carefully written library we can trace internal api calls or reimplement them on the outside.


--pancake