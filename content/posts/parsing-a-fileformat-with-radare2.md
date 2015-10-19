+++
date = "2015-01-11T01:55:32+01:00"
draft = false
title = "Parsing a fileformat with radare2"
slug = "parsing-a-fileformat-with-radare2"
aliases = [
	"parsing-a-fileformat-with-radare2"
]
+++
Thanks to [Skia]( http://libskia.so/ ), one of our [RSoC]( http://rada.re/rsoc/ ) participants, radare2 is now able to show structures, like headers, in a meaningful way.

### Usage
Lets see an example together (or watch the [video]( https://asciinema.org/a/12053 )):
```
$ r2 -nn /bin/true
```

The `-nn` option tells radare2 to load predefined binary structures.


```
[0x00000000]> pf.
pf.elf_header [16]z[2]E[2]Exqqqxwwwwww ident (elf_type)type (elf_machine)machine version entry phoff shoff flags ehsize phentsize phnum shentsize shnum shstrndx
pf.elf_phdr qqqqqqqq type offset vaddr paddr filesz memsz flags align
pf.elf_shdr xxqqqqxxqq name type flags addr offset size link info addralign entsize
```

The `pf.` command list all relevant structures for the current file. Time to see the header:

```
[0x00000000]> pf.elf_header @ elf_header
               ident : 0x00000000 = .ELF...
                type : 0x00000010 =  type (enum) = 0x2 ; ET_EXEC
             machine : 0x00000012 =  machine (enum) = 0x3e ; EM_X86_64
             version : 0x00000014 = 0x00000001
               entry : 0x00000018 = (qword) 0x00000000004013e2
               phoff : 0x00000020 = (qword) 0x0000000000000040
               shoff : 0x00000028 = (qword) 0x0000000000006320
               flags : 0x00000030 = 0x00000000
              ehsize : 0x00000034 = 0x0040
           phentsize : 0x00000036 = 0x0038
               phnum : 0x00000038 = 0x0009
           shentsize : 0x0000003a = 0x0040
               shnum : 0x0000003c = 0x001c
            shstrndx : 0x0000003e = 0x001b
[0x00000000]> 
```

Woohoo, it's working! Currently, radare2 has native support for ELF, PE, MACH0, ... Feel free to implement more ;)

### Implementation of a new format
This is what a ELF header looks like:

```
#define EI_NIDENT 16

typedef struct {
        unsigned char   e_ident[EI_NIDENT];
        Elf32_Half      e_type;
        Elf32_Half      e_machine;
        Elf32_Word      e_version;
        Elf32_Addr      e_entry;
        Elf32_Off       e_phoff;
        Elf32_Off       e_shoff;
        Elf32_Word      e_flags;
        Elf32_Half      e_ehsize;
        Elf32_Half      e_phentsize;
        Elf32_Half      e_phnum;
        Elf32_Half      e_shentsize;
        Elf32_Half      e_shnum;
        Elf32_Half      e_shtrndx;
} Elf32_Ehdr;
```

1. Look at the structure defined in .h or any valuable documentation about a file format
2. Convert each component type in pf symbol equivalent, for example first is `unsigned char e_ident[16]`:
  * `unsigned char` is the type that makes up C strings. Here we've got an array of 16 elements.
  * `e_ident` should contains the *Magic Number*: A numerical constant or text value used to identify a file format. In ELF, this magic number is a magic text (`ELF`), so we better have to display/parse it like a string (`z`).
  
```
[16]z e_ident
```

Set this new type in pf just with `pf.elf_header [16]z e_ident`

To try that new type and parse an elf :

1. Open an elf file: `r2 /bin/ls`
2. Do not forget to set the type: `pf.elf_header [16]z e_ident`
3. Run stored format at offset 0 of the elf file: `pf.elf_header @ 0`.

You can also check the [video]( http://asciinema.org/a/12045 )

* * *

The next element is an `Elf32_Half`, in the same .h we can see that `Elf32_Half` is a uint16_t or `unsigned short int`, this type is **w** (as in `word`) in pf format.

If we merge both previous elements we have: `[16]zw e_ident e_type`

* * *

### Nested structures

Once you have created a format, you can include it in an other one to make nested structures. Imagine you create a pixel format like this: `pf.pixel bbb red green blue`

You can now include this pixel in a row structure containing 32 pixel for a 32p width picture with `pf.row [32]? (pixel)example_name`

The same way you can define a header format, and a global image format containing first the header, and then all the row you need: `pf.image ?[42]? (header)my_header (row)my_rows`

* * *

Still with nested structure, we can also make format to display linked list or other complex data structures. You define a format containing a pointer to itself: `pf.elem i*? data (elem)next`. Running that format will print it until finding a null pointer.


```
[0x00000000]> pf.elem i*? data (elem)next
[0x00000000]> pf.elem
      data : 0x00000000 = 42
      next : (*0x10) struct<elem>
      ---- :
      data : 0x00000010 = 4000
      next : (*0x20) struct<elem>
      ---- :
      data : 0x00000020 = 66
      next : (*0x0) NULL
[0x00000000]> 
```

### Writing with `pf`
We can also use pf to write/replace content in a parsed struct. For example we can replace the parsed magic number using `=`:

```
[0x00000000]> pf.elf_header.e_ident=.ELO...
w .ELO... @ 0x00000000
```

r2 displays the command to type to overwrite the elf magic : `w .ELO... @ 0x00000000`

We can also directly prepend the pf command with . to write without need of copy/paste the write command.

```
. pf.elf_header.e_ident=.ELO...
```

There is also a [video]( http://asciinema.org/a/12138 ).

### Conclusion
Now you should be able to parse every file format you want with radare2! If you want to integrate your templates within the codebase, make sure to check the [wiki]( https://github.com/radare/radare2/wiki/Parse-a-file-format ) about how to do this.

We're currently working on improving the structures support, so stay tuned for more awesomenes!