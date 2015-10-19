+++
date = "2015-04-18T15:00:00+02:00"
draft = false
title = "Rop'n'roll"
slug = "ropnroll"
aliases = [
	"ropnroll"
]
+++
You may have already read this article 8 months ago, but since we changed a lot the ROP-related syntax, we're quite sure that you won't mind reading an updated version

As attackers are moving forwards, so does the defense. Since a couple of years, every decent operating system has non-executable stack, defeating the classic 'put your shellcode on the stack and execute it' *modus operanti*.

This is why attackers are now using (among other things) [Return Oriented Programming]( https://en.wikipedia.org/wiki/Return-oriented_programming ), also known as *ROP*, to bypass this protection.

Because radare2 (also) aims to be useful to exploits writers and make their life easier, it can:

- hunt for gadgets, with a configurable depth
- filter gadgets
- do this for multiples archs

```
[0x08048320]> /R
Do you want to print 468.9K chars? (y/N)
```

Well, let's filter.


```
[0x08048350]> /R call eax
  0x080483a3   0085c0741155  add byte [ebp + 0x551174c0], al
  0x080483a9           89e5  mov ebp, esp
  0x080483ab         83ec14  sub esp, 0x14
  0x080483ae     6824a00408  push 0x804a024
  0x080483b3           ffd0  call eax

  0x080483a8             55  push ebp
  0x080483a9           89e5  mov ebp, esp
  0x080483ab         83ec14  sub esp, 0x14
  0x080483ae     6824a00408  push 0x804a024
  0x080483b3           ffd0  call eax

  0x080483ac             ec  in al, dx
  0x080483ad           1468  adc al, 0x68
  0x080483af           24a0  and al, 0xa0
  0x080483b1           0408  add al, 8
  0x080483b3           ffd0  call eax

[0x08048350]> 

```

You can also display them in a linear manner, à la [ROPGadget]( https://github.com/JonathanSalwan/ROPgadget ), you just have to use `/Rl`:

```
[0x08048350]> /Rl leave
0x080483b3: call eax; add esp, 0x10; leave; ret;
0x080483b6: les edx, [eax]; leave; ret;
0x080483ed: call ecx; add esp, 0x10; leave; ret;
0x080483f0: les edx, [eax]; leave; ret;
0x0804840f: call 0x8048390; mov byte [0x804a024], 1; leave; ret;
0x08048419: or byte [ecx], al; leave; ret;
0x08048440: call edx; add esp, 0x10; leave; jmp 0x80483c0;
0x08048443: les edx, [eax]; leave; jmp 0x80483c0;
0x080484af: call 0x8048320; add esp, 0x10; mov edi, dword [ebp - 4]; leave; ret;
0x080484b3: inc dword [ebx + 0x7d8b10c4]; cld; leave; ret;
0x080484b5: les edx, [eax]; mov edi, dword [ebp - 4]; leave; ret;
0x080484b8: jge 0x80484b6; leave; ret;
0x080484ee: adc byte [eax], bh; mov ecx, dword [ebp - 4]; leave; lea esp, [ecx - 4]; ret;
0x080484ef: mov eax, 0; mov ecx, dword [ebp - 4]; leave; lea esp, [ecx - 4]; ret;
0x080484f2: add byte [eax], al; mov ecx, dword [ebp - 4]; leave; lea esp, [ecx - 4]; ret;
0x080484f5: dec ebp; cld; leave; lea esp, [ecx - 4]; ret;
```

If you're searching for a ret-to-reg gadget, you're only interested into `call reg` ones; you can do this with regexp, with `/R/`
```
[0x08048350]> /R/ call e[abcd]x
  0x080483a3   0085c0741155  add byte [ebp + 0x551174c0], al
  0x080483a9           89e5  mov ebp, esp
  0x080483ab         83ec14  sub esp, 0x14
  0x080483ae     6824a00408  push 0x804a024
  0x080483b3           ffd0  call eax

  0x080483a8             55  push ebp
  0x080483a9           89e5  mov ebp, esp
  0x080483ab         83ec14  sub esp, 0x14
  0x080483ae     6824a00408  push 0x804a024
  0x080483b3           ffd0  call eax

  0x080483ac             ec  in al, dx
  0x080483ad           1468  adc al, 0x68
  0x080483af           24a0  and al, 0xa0
  0x080483b1           0408  add al, 8
  0x080483b3           ffd0  call eax

  0x080483e2           89e5  mov ebp, esp
  0x080483e4         83ec10  sub esp, 0x10
  0x080483e7             50  push eax
  0x080483e8     6824a00408  push 0x804a024
  0x080483ed           ffd1  call ecx

  0x080483e5             ec  in al, dx
  0x080483e6         105068  adc byte [eax + 0x68], dl
  0x080483e9           24a0  and al, 0xa0
  0x080483eb           0408  add al, 8
  0x080483ed           ffd1  call ecx

  0x08048434   0085d274f255  add byte [ebp + 0x55f274d2], al
  0x0804843a           89e5  mov ebp, esp
  0x0804843c         83ec14  sub esp, 0x14
  0x0804843f             50  push eax
  0x08048440           ffd2  call edx

  0x08048436       d274f255  sal byte [edx + esi*8 + 0x55], cl
  0x0804843a           89e5  mov ebp, esp
  0x0804843c         83ec14  sub esp, 0x14
  0x0804843f             50  push eax
  0x08048440           ffd2  call edx

  0x08048438           f255  push ebp
  0x0804843a           89e5  mov ebp, esp
  0x0804843c         83ec14  sub esp, 0x14
  0x0804843f             50  push eax
  0x08048440           ffd2  call edx

  0x08048439             55  push ebp
  0x0804843a           89e5  mov ebp, esp
  0x0804843c         83ec14  sub esp, 0x14
  0x0804843f             50  push eax
  0x08048440           ffd2  call edx

  0x0804843b           e583  in eax, -0x7d
  0x0804843d             ec  in al, dx
  0x0804843e           1450  adc al, 0x50
  0x08048440           ffd2  call edx
```

Well, we only want the `call reg` part, we don't care about the previous operands:

```
[0x08048350]> /R/ call e[abcd]x~call
  0x080483b3           ffd0  call eax
  0x080483b3           ffd0  call eax
  0x080483b3           ffd0  call eax
  0x080483ed           ffd1  call ecx
  0x080483ed           ffd1  call ecx
  0x08048440           ffd2  call edx
  0x08048440           ffd2  call edx
  0x08048440           ffd2  call edx
  0x08048440           ffd2  call edx
  0x08048440           ffd2  call edx
```

Ho, by the way, if you want to search for several gadgets, the separator is `;`, which also happen to be the one to seperate commands in r2. This is why you need to quote the **whole** command like this:

```
[0x08048350]> "/R/ inc *;ret" 
  0x08048413           ffc6  inc esi
  0x08048415     0524a00408  add eax, 0x804a024
  0x0804841a           01c9  add ecx, ecx
  0x0804841c           f3c3  ret

  0x080484b3   ff83c4108b7d  inc dword [ebx + 0x7d8b10c4]
  0x080484b9             fc  cld
  0x080484ba             c9  leave
  0x080484bb             c3  ret

  0x0804867c             41  inc ecx
  0x0804867d             c3  ret

[0x08048350]> 
```

It's possible to change the depth of the search, to speed-up the hunt:
```
[0x08048320]> e search.roplen = 4
[0x08048320]> /R mov ebp,call eax
0x08048386 call eax
  0x0804837a         89e5  mov ebp, esp
  0x0804837c       83ec18  sub esp, 0x18
  0x0804837f c7042420a00408  mov dword [esp], 0x804a020
  0x08048386         ffd0  call eax

0x0804840f call eax
  0x08048403         89e5  mov ebp, esp
  0x08048405       83ec18  sub esp, 0x18
  0x08048408 c70424109f0408  mov dword [esp], 0x8049f10
  0x0804840f         ffd0  call eax

[0x08048320]>
```

And as always, you can append `j` to get a JSON output.

The next step might be ROP chain assembling and manipulation, and why not automatic-construction, à la [mona.py]( https://github.com/radare/radare2/issues/1019 )? Or maybe a semantic search powered by [ESIL](https://github.com/radare/radare2/wiki/ESIL )?

Anyway, contributions are welcome, and if you're using radare2 to pop somes shells, we'll be *delighted* to know about it!