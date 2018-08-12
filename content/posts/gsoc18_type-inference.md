+++
date = "2018-08-12T01:45:16+02:00"
draft = false
title = "GSoC'18 Final: Type inference"
slug = "type_inference"
aliases = [
	"Type inference"
]
+++

GSoC has almost been finished. I would like to summarize [my work](https://github.com/radare/radare2/commits?author=sivaramaaa) done so far this summer.

The goal of this task was to integrate types handling into the radare2 analysis loop, including automatic inference and suggestions.

### Example

* C - source code

```C
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

void main() {

    int length, length2;
    char *final;
    char *s1 = "Hello";
    char *s2 = " r2-folks";

    length = strlen (s1);
    length2 = strlen (s2);
}
```

* Before my work

```
/ (fcn) sym.main 157
|   sym.main ();
|           ; var int local_20h @ rbp-0x20
|           ; var int local_1ch @ rbp-0x1c
|           ; var int local_18h @ rbp-0x18
|           ; var int local_10h @ rbp-0x10
|           ; var int local_8h @ rbp-0x8
|           ; DATA XREF from entry0 (0x6bd)
|           0x000007aa      55             push rbp
|           0x000007ab      4889e5         mov rbp, rsp
|           0x000007ae      4883ec20       sub rsp, 0x20
|           0x000007b2      488d051b0100.  lea rax, str.Hello          ; 0x8d4 ; "Hello"
|           0x000007b9      488945e8       mov qword [local_18h], rax
|           0x000007bd      488d05160100.  lea rax, str.r2_folks       ; 0x8da ; " r2-folks"
|           0x000007c4      488945f0       mov qword [local_10h], rax
|           0x000007c8      488b45e8       mov rax, qword [local_18h]
|           0x000007cc      4889c7         mov rdi, rax
|           0x000007cf      e88cfeffff     call sym.imp.strlen         ; size_t strlen(const char *s)
|           0x000007d4      8945e0         mov dword [local_20h], eax
|           0x000007d7      488b45f0       mov rax, qword [local_10h]
|           0x000007db      4889c7         mov rdi, rax
|           0x000007de      e87dfeffff     call sym.imp.strlen         ; size_t strlen(const char *s)
```

* After

```
/ (fcn) sym.main 157
|   sym.main (int argc, char **argv, char **envp);
|           ; var size_t local_20h @ rbp-0x20
|           ; var size_t size @ rbp-0x1c
|           ; var char *src @ rbp-0x18
|           ; var char *s2 @ rbp-0x10
|           ; var char *dest @ rbp-0x8
|           ; DATA XREF from entry0 (0x6bd)
|           0x000007aa      55             push rbp
|           0x000007ab      4889e5         mov rbp, rsp
|           0x000007ae      4883ec20       sub rsp, 0x20
|           0x000007b2      488d051b0100.  lea rax, str.Hello          ; 0x8d4 ; "Hello"
|           0x000007b9      488945e8       mov qword [src], rax
|           0x000007bd      488d05160100.  lea rax, str.r2_folks       ; 0x8da ; " r2-folks"
|           0x000007c4      488945f0       mov qword [s2], rax
|           0x000007c8      488b45e8       mov rax, qword [src]
|           0x000007cc      4889c7         mov rdi, rax                ; const char *s
|           0x000007cf      e88cfeffff     call sym.imp.strlen         ; size_t strlen(const char *s)
|           0x000007d4      8945e0         mov dword [local_20h], eax
|           0x000007d7      488b45f0       mov rax, qword [s2]
|           0x000007db      4889c7         mov rdi, rax                ; const char *s
|           0x000007de      e87dfeffff     call sym.imp.strlen         ; size_t strlen(const char *s)
```


### Implementation

The type inference is well integrated with command `afta` and mainly implemented using these techniques : 

* Function signature

The arguments type and return type of known library functions are stored in sdb (you can inspect it using `k` comands), so whenever we hit a known function call while performing analysis, we extract that info and propagate that in both forward and backward manner. This step contributes the major portion of the type inference

* Instruction access pattern

1) Strings

```
lea rax, str.Hello
mov qword [local_30h], rax
```

When we have a load instruction with an address of string flagspace and then followed mov instruction having
a local variable as it's destination and string address as it's source, we can infer the type of local variable as `char *`

2) Signed and Unsigned

```
/ (fcn) main 202
|           ; var signed int local_34h @ rbp-0x34
|           ; var signed int local_30h @ rbp-0x30
|           ; var unsigned int local_28h @ rbp-0x28
|           0x000007a8      8945d0         mov dword [local_30h], eax
|       ,=< 0x000007ab      eb43           jmp 0x7f0
|    :|:|   0x000007cd      8b45dc         mov eax, dword [local_24h]
|    :|:|   0x000007d0      3b45cc         cmp eax, dword [local_34h]
|   ,=====< 0x000007d3      7e06           jle 0x7db
```

Instructions like `ja, jae, jb, jbe, je,jne` are typed as unsigned

Instructions like `jge, jnl, jng, jnge` are typed as signed

3) Pointer operation

```
mov rax, qword [ptr]
mov rax, qword [rax]
mov rdi, rax                ; char *s
call sym.imp.sprintf        ; int sprintf(char *s, const char *format, ...)
```

Here we can see that rax contains address of local var (ptr), which is the dereferenced once and value is stored in rax, using info from `sprintf` we can infer that rax contains address of string, so from instruction access pattern we could infer that local var (ptr) is double pointer i.e `char **`

4) Format string

```
; var char *local_18h @ rbp-0x18
mov rax, qword [local_18h]
mov rsi, rax
lea rdi, str.msg_:__s       ; 0x84f ; "msg : %s\n"
call sym.imp.printf
```

So whenever we encounter functions like printf/ sprintf, we try to parse it's format string and extract format char and then use that to infer corresponding variable's type.

The format specifications are extracted from `anal/d/spec.sdb`. You could create a new profile for specifying the set of format chars depending upon different libraries/operating systems/programming languages like this :

```
win=spec
spec.win.u32=unsigned int
```

Then change your default specification to newly created one using this config variable `e anal.spec = win`

### Future improvements

#### Constrained types

The idea here is to implement something like this : 

```
var int local_20h {0..10 | 30..40}
var char* local_30h {< 100}
```

This is also related to Radeco issues:

* Simple Type System in IR [#118](https://github.com/radareorg/radeco-lib/issues/118)
* Use Chalk engine for solving constraints and as a decision engine [#182](https://github.com/radareorg/radeco-lib/issues/182)

#### Further analysis

* Getting the function's argument values for each xrefs-to it [#10783](https://github.com/radare/radare2/issues/10783)

* Extract demangled parameter types and return type function args [#4756](https://github.com/radare/radare2/issues/4756)

#### DWARF/PDB integration

Load data types/structs from DWARF [#741](https://github.com/radare/radare2/issues/741)
