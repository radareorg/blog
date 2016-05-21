+++
date = "2014-07-02T02:27:59+02:00"
draft = false
title = "Types"
slug = "types"
aliases = [
	"types"
]
+++
One of the most wanted features for the RSoC was the support for 010-like templates. This is still planned, but there have been no recent movement on the topic.

But some of the basic cparse support has been implemented and I think it's time to get in touch with it in order to get ready for the integration with the rest of the analysis engine.

Current cparse is able to handle cpp and C syntax with support for enums, structs and nested structs. There's an ongoing discussion to define how function signatures should be stored in sdb.

Types are managed with the 't' command. By default it loads some basic ones like 'char *', 'unsigned int' or 'long'. The 't' command dumps the core->anal->sdb_types database which contains the following structure:

```
[0x00000000]> t
unsigned int=type
unsigned char=type
unsigned short=type
int=type
long=type
void *=type
char=type
char *=type
const char*=type
type.unsigned int=i
type.unsigned char=b
type.unsigned short=w
type.int=d
type.long=x
type.void *=p
type.char=x
type.char *=*z
type.const char*=*z
```

We can load a C include file with the 'to' command:

```
$ r2 -
[0x00000000]> cat test.h
#define uint32_t unsigned int
#define uint16_t unsigned short
#define uint8_t unsigned char

typedef struct name {
        char first[40];
        char middle[40];
        char last[40];
} name_t;

typedef struct date {
        uint8_t day;
        uint8_t month;
        uint16_t year;
} date_t;

typedef struct addr {
        char street[127];
        char city[40];
        uint32_t zip;
} addr_t;

typedef struct dox {
        addr_t address;
        name_t name;
        date_t bday;
} dox_t;
```

Note that the header have been tuned to rename some types to the ones supported by cparse (layer on top of libr_tcc).
```
[0x00000000]> to test.h
```

After this command the list of types found in 't' will be longer.

Displaying Types
------------
Types are linked to 'pf' print format command, so the types specify the format char of 'pf' to display or modify it.

Types can be mapped at any address with the 'tl' command, which will be used in analysis. Or 'tp' command can be used to cast your type and show the data in there. This is used by the disasm loop to display inline structures.

```
[0x00000000]> tl addr 0x4000
[0x00000000]> tp addr 0x4000
struct addr {
street : 0x00004000 = "Wallaby Way"
  city : 0x00004000 = "Sydney"
   zip : 0x00004008 = 2000
}
```

To display the command that is being executed the 't' command accepts a type argument which we may use to get the format string and field names:
```
[0x00000000]> t addr
pf *z*zi street city zip
```

Then we just use the magic of the r2 shell to interpret the command right there at the address we want:
```
[0x00000000]> .t addr @ 0x804840
```

Note the dot at the begining will execute every line of the output of the command as an r2 command. The '@' token, splits the line and evaluates the right expression. Then it does a temporal seek at this address runs the command and comes back.

There are also commands 'ts', 'tu' and 'te' - to show structure types, unions and enums
correspondigly.

If you want to learn more on Types. See the rest of the commands of 't?'. And if you feel brave don't esitate to checkout the source and send us a patch.

The future is cparse includes conditional structures and full write support for modifying struct fields.

--pancake
