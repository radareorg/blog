+++
date = "2015-12-08T01:00:00+02:00"
draft = false
title = "Unpacking shikata-ga-nai by scripting radare2"
slug = "unpacking-shikata-ga-nai-by-scripting-radare2"
aliases = [
    "unpacking-shikata-ga-nai-by-scripting-radare2"
]
+++

During latest [hacklu]( http://hack.lu )'s radare workshop, one part was
dedicated to *how to generically unpack shikata-ga-nai*. This blogpost is a
simple transposition of the slides into a blogpost.

Disclaimer: almost everything here is <s>stolen</s> based on ideas from
[NighterMan]( https://twitter.com/nighterman ).

First, was is Shitkata-ga-nai?
It's a polymorphic shellcode encoder implemented into [metasploit]( https://www.metasploit.com/ ):

```
msf > info encoder/x86/shikata_ga_nai > out.txt

       Name: Polymorphic XOR Additive Feedback Encoder
     Module: encoder/x86/shikata_ga_nai
   Platform: All
       Arch: x86
       Rank: Excellent

Provided by:
  spoonm <spoonm@no$email.com>

Description:
  This encoder implements a polymorphic XOR additive feedback encoder. 
  The decoder stub is generated based on dynamic instruction 
  substitution and dynamic block ordering. Registers are also selected 
  dynamically.
```

Looks scary. To unpack it, you have several solutions:

1. Run it on your machine and see what happens
2. Step-step-step-step-step-step in GDB
3. Trace the execution in a virtual machine
4. Use radare2 with *ESIL*.

ESIL (Evaluable String Intermediary Language) is radare2's own *Intermediary Language* (IL),
[RPN]( https://en.wikipedia.org/wiki/Reverse_Polish_notation )-ish, used for
emulation, [decompilation]( https://github.com/radare/radeco ), analysis, …

Now that we know that we can emulate the sample, the next step is to guess
where is the end of the packer, to dump the shellcode. Unfortunately, the
encoder is polymorphic, its blocks are permuted and registers are dynamically
selected.

If you take a look at its [source-code](
https://github.com/rapid7/metasploit-framework/blob/master/modules/encoders/x86/shikata_ga_nai.rb
), you can see that the last instruction will always be `loop`, so we can
emulate the sample until the last `loop` instruction, and then dump from here
to get the shellcode in clear.


Thanks to `r2pipe`, you can script radare2 with almost every popular language:

- `pip install r2pipe` for Python
- `gem install r2pipe` for ruby
- `npm install r2pipe` for nodejs
- …

But there is an issue: ESIL doesn't support [FPU]( https://en.wikipedia.org/wiki/Floating-point_unit )
instructions (yet), and `fnstenv` is [used to get EIP]( http://gynvael.coldwind.pl/n/eip_from_fpu_x86 )
(you should really read this article by the way).
Fortunately, this is the only use of FPU instruction by this encoder, so it's
trivial to emulated them by cheating.

We should make sure that radare2 is detecting every possible FPU opcode used as
FPU ones:

```python
import r2pipe
import sys

opcodes = [
'd9d0', 'd9e1', 'd9f6', 'd9f7', 'd9e5', 'd9e8', 'd9e9', 'd9ea', 'd9eb', 'd9ec',
'd9ed', 'd9c0', 'd9c1', 'd9c2', 'd9c3', 'd9c4', 'd9c5', 'd9c6', 'd9c7', 'd9c8',
'd9c9', 'd9ca', 'd9cb', 'd9cc', 'd9cd', 'd9ce', 'dac0', 'dac1', 'dac2', 'dac3',
'dac4', 'dac5', 'dac6', 'dac7', 'dac8', 'dac9', 'daca', 'dacb', 'dacc', 'dacd',
'dace', 'dacf', 'dad0', 'dad1', 'dad2', 'dad3', 'dad4', 'dad5', 'dad6', 'dad7',
'dad8', 'dad9', 'dada', 'dadb', 'dadc', 'dadd', 'dade', 'dbc0', 'dbc1', 'dbc2',
'dbc3', 'dbc4', 'dbc5', 'dbc6', 'dbc7', 'dbc8', 'dbc9', 'dbca', 'dbcb', 'dbcc',
'dbcd', 'dbce', 'dbcf', 'dbd0', 'dbd1', 'dbd2', 'dbd3', 'dbd4', 'dbd5', 'dbd6',
'dbd7', 'dbd8', 'dbd9', 'dbda', 'dbdb', 'dbdc', 'dbdd', 'dbde', 'ddc0', 'ddc1',
'ddc2', 'ddc3', 'ddc4', 'ddc5', 'ddc6'
]

r = r2pipe.open('-')


for op in opcodes:
    family = r.cmdj('abj %s' % op)[0]['family']
    if family != 'fpu':
        print 'NOT FPU: %s' % op
        sys.exit(0)
print 'End of the loop, every opcode was detected as FPU.'
```

If you launch this script with the latest version of radare2 installed on your
machine, you'll see that radare2 detects every opcode as FPU ; so far so good.
Feel free to add some dummy opcodes to see the detection fail by the way ;)

So now it's time to actually unpack shikata-ga-nai, for real. The plan is to:

1. Initialize the ESIL vm
2. If the instruction `invalid`:
  1. We're at the end of the shellcode!
  2. Dump from the last encountered `loop` instruction to the end
  3. Exit
3. Else, if the instruction is a `fpu` one:
  1. If it's `fnstenv`, write the previously stored `eip` at `esp`.
  2. Else, store `eip`
  3. Goto 1
4. Else, if the instruction is `loop`, store its location
5. Emulate the current instruction
6. Goto 1

This is the implementation in Python:

```python
import sys
import r2pipe

def initESIL():
    r.cmd('e io.cache=true')  # we're not really going to write anything anywhere
    r.cmd('e asm.bits=32')  # the shellcode is 32b
    r.cmd('e asm.arch=x86')
    r.cmd('aei')  # initialize the ESIL VM
    r.cmd('aeim 0xffffd000 0x2000 stack')
    r.cmd('.ar*')  # set all registers to zero
    r.cmd('aer esp=0xffffd000')
    r.cmd('aer ebp=0xffffd000')

def dump (start):
    end = r.cmdj('oj')[0]['size']  # size of the opened object

    print(r.cmd('pD %d @ %d' % (end-start, start)))  # disassembly

    raw = r.cmdj('p8j %d @ %d' % (end-start, start))  # dump
    with open('out', 'w') as f:
        f.write(''.join(map(chr, raw)))


def decode(r):
    lastfpu = 0
    lastloop = 0

    for i in range(100000):
        current_op = r.cmdj('pdj 1 @ eip')[0]

        # End of shellcode or invalid opcode
        if current_op['type'] == 'invalid':
            dump(lastloop)
            return

        # ESIL doesn't implement FPU (yet),
        # but we don't care, since they are only used
        # to get EIP with the FNSTENV technique
        # (http://gynvael.coldwind.pl/n/eip_from_fpu_x86).
        #
        # So we take note of the offset of the latest FPU instruction,
        # on put it in `esp` when `fnstenv` is encounted.
        if current_op['family'] == 'fpu':
            if current_op['opcode'].startswith('fnstenv'):
                r.cmd('wv %d @ esp' % lastfpu)
            else:
                lastfpu = current_op['offset']

        # Check for end of loop opcodes
        if current_op['opcode'].startswith('loop') and r.cmdj('arj')['ecx'] <= 1:
            lastloop = current_op['offset'] + current_op['size'];

        r.cmd('aes')

    print('[-] We emulated %d instructions, giving up' % i)


if len(sys.argv) != 2:
    print('[*] Usage: %s sample' % sys.argv[0])
    sys.exit(0)

r = r2pipe.open(sys.argv[1])
r.cmd('e asm.comments=false');
r.cmd('e asm.lines=false');
r.cmd('e asm.flags=false');
initESIL()
decode(r)
```

By the way, if you like this kind of writeup, feel free to take a look at
[jvoisin]( https://dustri.org/b/ )'s blog for more.
