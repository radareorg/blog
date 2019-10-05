+++
date = "2014-10-25T15:23:58+02:00"
draft = false
title = "Solving 'At gunpoint' from hack.lu 2014 with radare2"
slug = "solving-at-gunpoint-from-hack-lu-2014-with-radare2"
aliases = [
	"solving-at-gunpoint-from-hack-lu-2014-with-radare2"
]
+++
Many thanks to [crowell]( https://twitter.com/jeffreycrowell ) for giving us the permission to publish his writeup on this blog. Feel also free to take a look at depierre's [one]( http://depier.re/hacklu_2k14_gunpoint_200/ ).

"At Gunpoint" was a 200 point Reversing challenge in Hack.lu ctf 2014. The description is as follows

```
You 're the sheriff of a small town, investigating news about a gangster squad
passing by. Rumor has it they' re easy to outsmart, so you have just followed
one to their encampment by the river.You know you can easily take them out one
by one, if you would just know their secret handshake..
```

Let's see what we are working with.

```
minishwoods hacklu/gunpoint» file gunpoint_2daf5fe3fb236b398ff9e5705a058a7f.dat
gunpoint_2daf5fe3fb236b398ff9e5705a058a7f.dat: Gameboy ROM: "FLUX", [ROM ONLY], ROM: 256Kbit
```

So it is a Gameboy ROM. I load it up in [VisualBoyAdvance]( http://sourceforge.net/projects/vba/ ) and see what the game is.

![running_game_1.png](/blog/images/pVD3Je8-1.png)

It is a static image, and after some time, we receive this text.

![running_game_2.png](/blog/images/meh.png)

Looks like we will have to put in some inputs from the joypad to get the flag. It is a crackme style but for the Gameboy! Sounds fun.

First thing to do is to find out how the Gameboy interprets button presses.

According to this document [here]( http://www.semis.demon.co.uk/Gameboy/Gbspec.txt ), the joypad is memory mapped to `0xFF00`. So now that we have some sort of idea of what we are looking for, let's load up the ROM in radare2!

First, let's run the auto analysis and see what r2 can give us for functions. We run `aa` to analyze the binary, then `afl` to list the functions.

```
minishwoods hacklu/gunpoint » radare2 ./gunpoint_2daf5fe3fb236b398ff9e5705a058a7f.dat
 -- Fuck you, fuck you in the mouth, with a chair!
 [0x00000100]> aa
 [0x00000100]> afl
 0x00000100  35  1  entry0
 0x00000150  133  8  main
 0x00000123  178  8  fcn.00000123
```

Hm... not so great, just 3 functions. (note, since after the ctf ended, the latest git of r2 is much better at detecting Gameboy functions!)

Let's try to find usage of the joypad. Really what we are looking for is a load or store from `0xFF00`, but I am lazy, and the ROM is small, so we can just disassembly everything and grep for it!

```asm
[0x00000100]> s 0; pd 10000 | grep ff00
Do you want to print 566.8K chars? (y/N)
|           0x00000193    e2           ld [0xff00 + c], a
            0x00000c84    f000         ld a, [0xff00]
|           0x00001e63    e000         ld [0xff00], a
|           0x00001e65    f000         ld a, [0xff00]
|           0x00001e67    f000         ld a, [0xff00]
|           0x00001e71    e000         ld [0xff00], a
|           0x00001e73    f000         ld a, [0xff00]
|           0x00001e75    f000         ld a, [0xff00]
|           0x00001e77    f000         ld a, [0xff00]
|           0x00001e79    f000         ld a, [0xff00]
|           0x00001e7b    f000         ld a, [0xff00]
|           0x00001e7d    f000         ld a, [0xff00]
|           0x00001e88    e000         ld [0xff00], a
            0x00002307    e2           ld [0xff00 + c], a
            0x00002317    f2           ld a, [0xff00 + c]
            0x0000233f    f2           ld a, [0xff00 + c]
            0x00002342    f2           ld a, [0xff00 + c]
            0x00002717    f2           ld a, [0xff00 + c]
```

Perfect, not many uses, and it looks like there are a few small groups that are touching `0xFF00`, so probably just a few functions that interact with the joypad.

So, seek to 0x1e63 (`s 0x1e63`), then open up visual mode with `V`. Scroll up a bit to see that at `0x1e5f` is a `ret`, then `0x1e60` is `push bc`. The push instruction looks to be the start of a function that interacts with the joypad quite a bit. So we can define this as a function from visual mode with `df`. (If you have a newer version of r2, this is probably already done for you).

Checking back at the Gameboy document, The function looks very similar, in fact, it nearly identical to the joypad reader from Ms. PacMan.

Code from Ms. PacMan is here

```asm
       Example code:

          Game: Ms. Pacman
          Address: $3b1

        LD A,$20       <- bit 5 = $20
        LD ($FF00),A   <- select P14 by setting it low
        LD A,($FF00)
        LD A,($FF00)   <- wait a few cycles
        CPL            <- complement A
        AND $0F        <- get only first 4 bits
        SWAP A         <- swap it
        LD B,A         <- store A in B
        LD A,$10
        LD ($FF00),A   <- select P15 by setting it low
        LD A,($FF00)
        LD A,($FF00)
        LD A,($FF00)
        LD A,($FF00)
        LD A,($FF00)
        LD A,($FF00)   <- Wait a few MORE cycles
        CPL            <- complement (invert)
        AND $0F        <- get first 4 bits
        OR B           <- put A and B together

        LD B,A         <- store A in D
        LD A,($FF8B)   <- read old joy data from ram
        XOR B          <- toggle w/current button bit
        AND B          <- get current button bit back
        LD ($FF8C),A   <- save in new Joydata storage
        LD A,B         <- put original value in A
        LD ($FF8B),A   <- store it as old joy data


        LD A,$30       <- deselect P14 and P15
        LD ($FF00),A   <- RESET Joypad
        RET            <- Return from Subroutine

          The button values using the above method are such:
          $80 - Start             $8 - Down
          $40 - Select            $4 - Up
          $20 - B                 $2 - Left
          $10 - A                 $1 - Right

          Let's say we held down A, Start, and Up.
          The value returned in accumulator A would be $94jkj
```

And the code we have here.

```asm
/ (fcn) fcn.00001e60 45
|          0x00001e60    c5           push bc
|          0x00001e61    3e20         ld a, 0x20
|          ;  JOYPAD
|          0x00001e63    e000         ld [0xff00], a
|          ;  JOYPAD
|          0x00001e65    f000         ld a, [0xff00]
|          ;  JOYPAD
|          0x00001e67    f000         ld a, [0xff00]
|          0x00001e69    2f           cpl
|          0x00001e6a    e60f         and 0x0f
|          0x00001e6c    cb37         swap a
|          0x00001e6e    47           ld b, a
|          0x00001e6f    3e10         ld a, 0x10
|          ;  JOYPAD
|          0x00001e71    e000         ld [0xff00], a
|          ;  JOYPAD
|          0x00001e73    f000         ld a, [0xff00]
|          ;  JOYPAD
|          0x00001e75    f000         ld a, [0xff00]
|          ;  JOYPAD
|          0x00001e77    f000         ld a, [0xff00]
|          ;  JOYPAD
|          0x00001e79    f000         ld a, [0xff00]
|          ;  JOYPAD
|          0x00001e7b    f000         ld a, [0xff00]
|          ;  JOYPAD
|          0x00001e7d    f000         ld a, [0xff00]
|          0x00001e7f    2f           cpl
|          0x00001e80    e60f         and 0x0f
|          0x00001e82    b0           or b
|          0x00001e83    cb37         swap a
|          0x00001e85    47           ld b, a
|          0x00001e86    3e30         ld a, 0x30
|          ;  JOYPAD
|          0x00001e88    e000         ld [0xff00], a
|          0x00001e8a    78           ld a, b
|          0x00001e8b    c1           pop bc
\          0x00001e8c    c9           ret
```

Looks really similar, doesn't it? ;) So we can see here, that value of the joypad is now stored in register A from `0x1e8a`. Seek to the beginning of the function and define the function with a name like `read_joypad` with `dr` in visual mode.

So, now that we know what reads the joypad, we can find the xrefs to find out the checking routine. From visual mode, at the top of `read_joypad`, hit `x` to see the xrefs.

```
[GOTO XREF]> 65 ./gunpoint_2daf5fe3fb236b398ff9e5705a058a7f.dat]> pd $r @ read_joypad
[0] 0x00001e60 CODE (CALL) XREF 0x00001e54 (fcn.00001e50)
[1] 0x00001e60 CODE (CALL) XREF 0x00001e8d (fcn.00001e8d)
[2] 0x00001e60 CODE (CALL) XREF 0x00001e94 (fcn.00001e94)
```

We see 3 cross references, which means we have 3 other functions calling the `read_joypad` function.

Looking at xref2, we see that the joypad is read, and then the value is also stored into register `e`

```asm
/ (fcn) load_joypad_to_e 5
|          0x00001e94    cd601e       call read_joypad ;[1]
|             read_joypad(0x0, 0x0, 0x0)
|          0x00001e97    5f           ld e, a
\          0x00001e98    c9           ret
```

Essentially, in pseudocode

```c
a = read_joypad();
e = a;
```

For some time I was stuck here, as it didn't seem that the `load_joypad_to_e` was called anywhere, r2 did not find any calls, but we can use the same trick we used to find the references to the memory mapped io of the joypad to find references to the `load_joypad_to_e`.

```
[0x00001e94]> s 0; pd 10000 | grep load_joypad
Do you want to print 566.8K chars? (y/N)
  --------> 0x00000d6f    cd941e       call load_joypad_to_e
          >    load_joypad_to_e()
  / (fcn) load_joypad_to_e 5
```

Looks like it is referenced just once, so seek to `0xd6f` and open visual mode. Looks to be the beginning of a big block of code, define as a function and start reversing. I defined it as `checker_loop` once I realized that this is the main crackme part.

It looks like the auto analysis fails to detect the proper end of the function. This is fine, because we can tell r2 exactly how long the function is ourselves.

Looking at the code, the last instruction of the function is the `ret` at `0xf01`, so, by starting at `0xd6f` the function should be 403 bytes. Just
define the function like this

`[0x00000e5f]> af+ 0xd6f 403 checker_loop`

Now we can disassemble the entire function with `pdf @ checker_loop`.

```asm
/ (fcn) checker_loop 403
|           0x00000d6f    cd941e       call load_joypad_to_e
|              load_joypad_to_e()
|           0x00000d72    21a2c0       ld hl, 0xc0a2
|           0x00000d75    73           ld [hl], e
|           0x00000d76    21a1c0       ld hl, 0xc0a1
|           0x00000d79    7e           ld a, [hl]
|           0x00000d7a    21a2c0       ld hl, 0xc0a2
|           0x00000d7d    be           cp [hl]
|       ,=< 0x00000d7e    2003         jr nZ, 0x03
|      ,==< 0x00000d80    c3f60e       jp loc.00000ef6
|      |`-> 0x00000d83    21a0c0       ld hl, 0xc0a0
|      |    0x00000d86    7e           ld a, [hl]
|      |    0x00000d87    fe0d         cp 0x0d
|     ,===< 0x00000d89    c2990d       jp nZ, 0x0d99
|    ,====< 0x00000d8c    1803         jr 0x03 ; (checker_loop)
|   ,=====< 0x00000d8e    c3990d       jp 0x0d99 ; (checker_loop)
|   |`----> 0x00000d91    21a0c0       ld hl, 0xc0a0
|   | ||    0x00000d94    3600         ld [hl], 0x00
|  ,======< 0x00000d96    c38a0e       jp loc.00000e8a
|  |`-`---> 0x00000d99    21a2c0       ld hl, 0xc0a2
|  |   |    0x00000d9c    7e           ld a, [hl]
|  |   |    0x00000d9d    e604         and 0x04
| ,=======< 0x00000d9f    2003         jr nZ, 0x03
| ========< 0x00000da1    c3c90d       jp loc.00000dc9
| `-------> 0x00000da4    21a0c0       ld hl, 0xc0a0
|  |   |    0x00000da7    7e           ld a, [hl]
|  |   |    0x00000da8    b7           or a
| ========< 0x00000da9    caba0d       jp Z, 0x0dba
|  |   |    0x00000dac    21a0c0       ld hl, 0xc0a0
|  |   |    0x00000daf    7e           ld a, [hl]
|  |   |    0x00000db0    fe04         cp 0x04
| ========< 0x00000db2    c2c10d       jp nZ, 0x0dc1
| ========< 0x00000db5    1803         jr 0x03 ; (checker_loop)
| ========< 0x00000db7    c3c10d       jp 0x0dc1 ; (checker_loop)
| --------> 0x00000dba    21a0c0       ld hl, 0xc0a0
|  |   |    0x00000dbd    34           inc [hl]
| ========< 0x00000dbe    c38a0e       jp loc.00000e8a
| --------> 0x00000dc1    21a0c0       ld hl, 0xc0a0
|  |   |    0x00000dc4    3600         ld [hl], 0x00
| ========< 0x00000dc6    c38a0e       jp loc.00000e8a
|- loc.00000dc9 49
| --------> 0x00000dc9    21a2c0       ld hl, 0xc0a2
|  |   |    0x00000dcc    7e           ld a, [hl]
|  |   |    0x00000dcd    e601         and 0x01
| ========< 0x00000dcf    2003         jr nZ, 0x03
| ========< 0x00000dd1    c3fa0d       jp loc.00000dfa
| --------> 0x00000dd4    21a0c0       ld hl, 0xc0a0
|  |   |    0x00000dd7    7e           ld a, [hl]
|  |   |    0x00000dd8    fe01         cp 0x01
| ========< 0x00000dda    caeb0d       jp Z, 0x0deb
|  |   |    0x00000ddd    21a0c0       ld hl, 0xc0a0
|  |   |    0x00000de0    7e           ld a, [hl]
|  |   |    0x00000de1    fe05         cp 0x05
| ========< 0x00000de3    c2f20d       jp nZ, 0x0df2
| ========< 0x00000de6    1803         jr 0x03 ; (loc.00000dc9)
| ========< 0x00000de8    c3f20d       jp 0x0df2 ; (loc.00000dc9)
| --------> 0x00000deb    21a0c0       ld hl, 0xc0a0
|  |   |    0x00000dee    34           inc [hl]
| ========< 0x00000def    c38a0e       jp loc.00000e8a
| --------> 0x00000df2    21a0c0       ld hl, 0xc0a0
|  |   |    0x00000df5    3600         ld [hl], 0x00
\ ========< 0x00000df7    c38a0e       jp loc.00000e8a
|- loc.00000dfa 49
| --------> 0x00000dfa    21a2c0       ld hl, 0xc0a2
|  |   |    0x00000dfd    7e           ld a, [hl]
|  |   |    0x00000dfe    e608         and 0x08
| ========< 0x00000e00    2003         jr nZ, 0x03
| ========< 0x00000e02    c32b0e       jp loc.00000e2b
| --------> 0x00000e05    21a0c0       ld hl, 0xc0a0
|  |   |    0x00000e08    7e           ld a, [hl]
|  |   |    0x00000e09    fe02         cp 0x02
| ========< 0x00000e0b    ca1c0e       jp Z, 0x0e1c
|  |   |    0x00000e0e    21a0c0       ld hl, 0xc0a0
|  |   |    0x00000e11    7e           ld a, [hl]
|  |   |    0x00000e12    fe06         cp 0x06
| ========< 0x00000e14    c2230e       jp nZ, 0x0e23
| ========< 0x00000e17    1803         jr 0x03 ; (loc.00000dfa)
| ========< 0x00000e19    c3230e       jp 0x0e23 ; (loc.00000dfa)
| --------> 0x00000e1c    21a0c0       ld hl, 0xc0a0
|  |   |    0x00000e1f    34           inc [hl]
| ========< 0x00000e20    c38a0e       jp loc.00000e8a
| --------> 0x00000e23    21a0c0       ld hl, 0xc0a0
|  |   |    0x00000e26    3600         ld [hl], 0x00
\ ========< 0x00000e28    c38a0e       jp loc.00000e8a
|- loc.00000e2b 49
| --------> 0x00000e2b    21a2c0       ld hl, 0xc0a2
|  |   |    0x00000e2e    7e           ld a, [hl]
|  |   |    0x00000e2f    e602         and 0x02
| ========< 0x00000e31    2003         jr nZ, 0x03
| ========< 0x00000e33    c35c0e       jp loc.00000e5c
| --------> 0x00000e36    21a0c0       ld hl, 0xc0a0
|  |   |    0x00000e39    7e           ld a, [hl]
|  |   |    0x00000e3a    fe03         cp 0x03
| ========< 0x00000e3c    ca4d0e       jp Z, 0x0e4d
|  |   |    0x00000e3f    21a0c0       ld hl, 0xc0a0
|  |   |    0x00000e42    7e           ld a, [hl]
|  |   |    0x00000e43    fe07         cp 0x07
| ========< 0x00000e45    c2540e       jp nZ, 0x0e54
| ========< 0x00000e48    1803         jr 0x03 ; (loc.00000e2b)
| ========< 0x00000e4a    c3540e       jp 0x0e54 ; (loc.00000e2b)
| --------> 0x00000e4d    21a0c0       ld hl, 0xc0a0
|  |   |    0x00000e50    34           inc [hl]
| ========< 0x00000e51    c38a0e       jp loc.00000e8a
| --------> 0x00000e54    21a0c0       ld hl, 0xc0a0
|  |   |    0x00000e57    3600         ld [hl], 0x00
\ ========< 0x00000e59    c38a0e       jp loc.00000e8a
|- loc.00000e5c 95
| --------> 0x00000e5c    21a2c0       ld hl, 0xc0a2
|  |   |    0x00000e5f    7e           ld a, [hl]
|  |   |    0x00000e60    e610         and 0x10
| ========< 0x00000e62    2003         jr nZ, 0x03
| ========< 0x00000e64    c38a0e       jp loc.00000e8a
| --------> 0x00000e67    21a0c0       ld hl, 0xc0a0
|  |   |    0x00000e6a    7e           ld a, [hl]
|  |   |    0x00000e6b    fe0a         cp 0x0a
| ========< 0x00000e6d    ca7e0e       jp Z, 0x0e7e
|  |   |    0x00000e70    21a0c0       ld hl, 0xc0a0
|  |   |    0x00000e73    7e           ld a, [hl]
|  |   |    0x00000e74    fe0b         cp 0x0b
| ========< 0x00000e76    c2850e       jp nZ, 0x0e85
| ========< 0x00000e79    1803         jr 0x03 ; (loc.00000e5c)
| ========< 0x00000e7b    c3850e       jp 0x0e85 ; (loc.00000e5c)
| --------> 0x00000e7e    21a0c0       ld hl, 0xc0a0
|  |   |    0x00000e81    34           inc [hl]
| ========< 0x00000e82    c38a0e       jp loc.00000e8a
| --------> 0x00000e85    21a0c0       ld hl, 0xc0a0
|  |   |    0x00000e88    3600         ld [hl], 0x00
|  |        ; XREFS: JMP 0x00000dbe  JMP 0x00000dc6  JMP 0x00000def
|  |        ; XREFS: JMP 0x00000df7  JMP 0x00000e20  JMP 0x00000e28
|  |        ; XREFS: JMP 0x00000e51  JMP 0x00000e59  JMP 0x00000e82
|  |        ; XREFS: JMP 0x00000e64
|- loc.00000e8a 49
| -`------> 0x00000e8a    21a2c0       ld hl, 0xc0a2
|      |    0x00000e8d    7e           ld a, [hl]
|      |    0x00000e8e    e620         and 0x20
| ========< 0x00000e90    2003         jr nZ, 0x03
| ========< 0x00000e92    c3bb0e       jp loc.00000ebb
| --------> 0x00000e95    21a0c0       ld hl, 0xc0a0
|      |    0x00000e98    7e           ld a, [hl]
|      |    0x00000e99    fe08         cp 0x08
| ========< 0x00000e9b    caac0e       jp Z, 0x0eac
|      |    0x00000e9e    21a0c0       ld hl, 0xc0a0
|      |    0x00000ea1    7e           ld a, [hl]
|      |    0x00000ea2    fe09         cp 0x09
| ========< 0x00000ea4    c2b30e       jp nZ, 0x0eb3
| ========< 0x00000ea7    1803         jr 0x03 ; (loc.00000e8a)
| ========< 0x00000ea9    c3b30e       jp 0x0eb3 ; (loc.00000e8a)
| --------> 0x00000eac    21a0c0       ld hl, 0xc0a0
|      |    0x00000eaf    34           inc [hl]
| ========< 0x00000eb0    c3f60e       jp loc.00000ef6
| --------> 0x00000eb3    21a0c0       ld hl, 0xc0a0
|      |    0x00000eb6    3600         ld [hl], 0x00
\ ========< 0x00000eb8    c3f60e       jp loc.00000ef6
|- loc.00000ebb 43
| --------> 0x00000ebb    21a2c0       ld hl, 0xc0a2
|      |    0x00000ebe    7e           ld a, [hl]
|      |    0x00000ebf    e680         and 0x80
| ========< 0x00000ec1    2003         jr nZ, 0x03
| ========< 0x00000ec3    c3e60e       jp loc.00000ee6
| --------> 0x00000ec6    21a0c0       ld hl, 0xc0a0
|      |    0x00000ec9    7e           ld a, [hl]
|      |    0x00000eca    fe0c         cp 0x0c
| ========< 0x00000ecc    c2de0e       jp nZ, 0x0ede
| ========< 0x00000ecf    1803         jr 0x03 ; (loc.00000ebb)
| ========< 0x00000ed1    c3de0e       jp 0x0ede ; (loc.00000ebb)
| --------> 0x00000ed4    cd0002       call 0x0200
|         >    0x00000200() ; main+176
|      |    0x00000ed7    21a0c0       ld hl, 0xc0a0
|      |    0x00000eda    34           inc [hl]
| ========< 0x00000edb    c3f60e       jp loc.00000ef6
| --------> 0x00000ede    21a0c0       ld hl, 0xc0a0
|      |    0x00000ee1    3600         ld [hl], 0x00
\ ========< 0x00000ee3    c3f60e       jp loc.00000ef6
|- loc.00000ee6 27
| --------> 0x00000ee6    21a2c0       ld hl, 0xc0a2
|      |    0x00000ee9    7e           ld a, [hl]
|      |    0x00000eea    e640         and 0x40
| ========< 0x00000eec    2003         jr nZ, 0x03
| ========< 0x00000eee    c3f60e       jp loc.00000ef6
| --------> 0x00000ef1    21a0c0       ld hl, 0xc0a0
|      |    0x00000ef4    3600         ld [hl], 0x00
|- loc.00000ef6 11
| -----`--> 0x00000ef6    21a2c0       ld hl, 0xc0a2
|           0x00000ef9    7e           ld a, [hl]
|           0x00000efa    21a1c0       ld hl, 0xc0a1
|           0x00000efd    77           ld [hl], a
\           0x00000efe    c36f0d       jp checker_loop
\           0x00000f01    c9           ret
```

Yikes! This is a long function. With r2 we have a lot of tools at our disposal though, and we can use the graph view to get a better high-level understanding of the code.

We can load up a graph visualization with either `VV` for an ascii view, or a graphviz view with `ag`. I like to use the graphviz output with xdot.

`[0x00000d6f]> ag $$ | xdot`

And we get a nice graph like this.  I have some annotations on mine, from reversing the function myself. This is helpful for looking at the overall way the function works. It seems to have a cascading check, like

`if(){}else if{}else if{}else{}`

type structure, which makes sense for value and index checking.

![biggraph.png](/blog/images/bg.png)

So, after some analysis, and guessing, we can figure the following. There is some debouncing/not expected to be perfect for clock cycles for button presses, so there must be tolerance for `0xFF00` to be the same for multiple cycles, or to be 0 with no penalty.

This means that the previous value must be stored somewhere, and that checks for empty press is somewhere too. From looking at the assembly, and the graph, I can piece together this as pseudocode.

```c
void main_checker() {
  byte reg_E;
  byte reg_A;
  byte* curr_press = 0xc0a2;
  byte* prev_press = 0xc0a1;
  byte* num_correct = 0xc0a0;
begin:
  reg_E = load_joypad_to_e();
  *curr_press = reg_E;
  reg_A = *prev_press;
  if (*curr_press == *prev_press) {
    goto begin;
  }
  if (*curr_press == 0x00) {
    goto begin;
  }
  do {
    if (*curr_press == 0x01) {
      if (*num_correct == 1 || *num_correct == 5) {
        *num_correct++;
      } else {
        *num_correct = 0;
      }
    } else if (*curr_press == 0x02) {
      if (*num_correct == 3 || num_correct == 7) {
        *num_correct++;
      } else {
        *num_correct = 0;
      }
    } else if (*curr_press == 0x04) {
      if (*num_correct == 0 || *num_correct == 4) {
        *num_correct++;
      } else {
        *num_correct = 0;
      }
    } else if (*curr_press == 0x08) {
      if (*num_correct == 2 || *num_correct == 6) {
        *num_correct++;
      } else {
        *num_correct = 0;
      }
    } else if (*curr_press == 0x10) {
      if (*num_correct == 10 || *num_correct == 11) {
        *num_correct++;
      } else {
        *num_correct = 0;
      }
    } else if (*curr_press == 0x20) {
      if (*num_correct == 8 || *num_correct == 9) {
        *num_correct++;
      } else {
        *num_correct = 0;
      }
    } else if (*curr_press == 0x80) {
      if (*num_correct == 12) {
        *num_correct++;
      } else {
        *num_correct = 0;
      }
    } else {
      *num_correct = 0;
    }
	*prev_press = *curr_press;
  } while (*num_correct != 0x0D);
  winner();
}

```

The only thing left to do is to cross reference the button presses with their values, and extract the order from that code.

```
4 - up
1 - right
8 - down
2 - left
4 - up
1 - right
8 - down
2 - left
20 - B
20 - B
10 - A
10 - A
80 - Start
---------values-----------------
$80 - Start             $8 - Down
$40 - Select            $4 - Up
$20 - B                 $2 - Left
$10 - A                 $1 - Right
```

Plug it in to VisualBoyAdvance and grab our 200 points!

![win.png](/blog/images/win.png)

Thanks to FluxFingers for a great CTF!

Comments? Questions?
`crowell@shellphish.net`
