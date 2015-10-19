+++
date = "2014-11-05T22:16:46+01:00"
draft = false
title = "Zignatures"
slug = "zignatures"
aliases = [
	"zignatures"
]
+++
by http://twitter.com/j0sm1

In this blog post, we are going to show a simple example of the radare2 “zignatures” functionality. To manage “zignatures” in radare2, the only thing you have to do is type ‘z’ in the radare console. Here you can get more info on this 'z' command:

    r2console> z?
    |Usage: z[abcp/*-] [arg]Zignatures
    | z                 show status of zignatures
    | z*                display all zignatures
    | z-prefix          unload zignatures with corresponding prefix
    | z-*               unload all zignatures
    | z/[ini] [end]     search zignatures between these regions
    | za ...            define new zignature for analysis
    | zb name bytes     define zignature for bytes
    | zc @ fcn.foo      flag signature if matching (.zc@@fcn)
    | zf name fmt       define function zignature (fast/slow, args, types)
    | zg prefix [file]  generate signature for current file
    | zh name bytes     define function header zignature
    | zp prefix         define prefix for following zignatures
    | zp                display current prefix
    | zp-               unset prefix
    | NOTE:             bytes can contain '.' (dots) to specify a binary mask

To show this functionality, we are going to use Go programming language[1]. Our example prints its process ID (pid.go) and then exits. You can see its code here:

    package main
    import "fmt"
    import "syscall"
    func main() {
        fmt.Println("My PID:")
        pid := syscall.Getpid()
        fmt.Printf("%d\n",pid) 
    }

First, we are going to compile this example statically and without stripping the compiled code:

    $ go build -compiler gccgo --gccgoflags -static pid.go
    $ r2 pid
    ## We can check if the binary is stripped with this r2 command
    > i~strip
    strip false

On the other hand, we compiled statically this same example with name pid2, and this time we also executed the strip command to delete information from this binary (symbols, etc.):

    $ go build -compiler gccgo --gccgoflags -static pid2.go
    $ strip pid2
    $ r2 pid2

We can check if the binary is stripped with this r2 command

    > i~strip
    $ strip true

Now, we create the “zignatures” from our static and unstripped binary (pid). After, we will apply this “zignatures” over the stripped binary (pid2).

We create “zignature” file go.z for Go programming language, with command ‘zg’.

    $ r2 pid
    > zg go go.z 

It’s time to open our stripped binary and watch which information is available. Load stripped binary with auto analysis:

    $ r2 -A pid2 

You can see that the stripped binary has no symbols, making our analysis more difficult. This situation is very common in CTFs. To recover some symbols from the binary we can use the zignatures we just created. Then we load the zignatures file:

    > . go.z

To show the zignatures loaded use the 'z' command.

    > z
    Loaded 4702 signatures
      4702 byte signatures
      0 head signatures
      0 func signatures

We look for the text section

    > iS~text 
    idx=05 vaddr=0x08048310 paddr=0x00000310 sz=1074039 vsz=1074039 perm=-r-x name=.text

We look for zignatures matching pid2's text section and r2 loads the matching zignatures

    > .z/ section..text section..text+1074039

After that, we are able to identify in the stripped binary, for example, the symbol of Go function fmt.Println:

    > aa
    > pd 35 @ sign.sign.b.sym.fmt.Println 
    |          ; CALL XREF from 0x08049adb (fcn.08049a6c)
    / (fcn) fcn.0805b030 96

You can see that radare2 shows a XREF for the symbol “sign.sign.b.sym.Println” (CALL XREF from 0x08049adb) in address 0x08049adb. In that address we have this code (nice! the symbol is identified):

    r2-(pid2)> pd 35 @ 0x08049adb-10
    |          0x08049ad1   push dword     [ebp-0x44]
    |          0x08049ad4   push dword [ebp-0x48]
    |          0x08049ad7   push dword [ebp-0x4c]
    |          0x08049ada   push eax
    |          0x08049adb   call fcn.0805b030
    |             fcn.0805b030(unk, unk, unk, unk) ; sign.sign.b.sym.fmt.Println
    |          0x08049ae0   add esp, 0xc
    |          0x08049ae3   call fcn.08095580

If you see this address in the unstripped binary (pid).

    r2-(pid)> pd 35 @ 0x08049adb-10
    |          0x08049ad1   push dword [ebp-0x44]
    |          0x08049ad4   push dword [ebp-0x48]
    |          0x08049ad7   push dword [ebp-0x4c]
    |          0x08049ada   push eax
    |          0x08049adb   call sym.fmt.Println
    |             sym.fmt.Println(unk, unk, unk, unk, unk, unk, unk)

Finally, if you don't load the zignatures in the stripped binary the result is as follows:

    r2-(pid2)> pd 35 @ 0x08049adb-10
    |          0x08049ad1   push dword [ebp-0x44]
    |          0x08049ad4   push dword [ebp-0x48]
    |          0x08049ad7   push dword [ebp-0x4c]
    |          0x08049ada   push eax
    |          0x08049adb   call fcn.0805b030
    |             fcn.0805b030(unk, unk, unk, unk, unk, unk, unk)

In this post we have seen how zignatures can help us to make the analysis of stripped binaries easier.

[1] https://golang.org/
