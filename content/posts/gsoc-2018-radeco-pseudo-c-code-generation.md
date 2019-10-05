+++
date = "2018-08-12T23:26:15+09:00"
draft = false
title = "Gsoc 2018 Radeco Pseudo C Code Generation"
slug = "gsoc_2018_radeco_pseudo_c_code_generation"
aliases = [
	"gsoc 2018 radeco code generation"
]
+++
# GSoC 2018 Radeco Pseudo C Code Generation

## Introduction

This summer, I was working on C-like pseudocode generation with radeco. Althrough radeco was able to analyze executables, it could not decompile analyzed executables. My work allows to use radeco for generating C-like pseudocode from analyzed executables.

### Usage

#### Installation
**Note:** Nightly Rust is required. You can install it using [rustup](https://rustup.rs/).
```shell
$ rustup install nightly
$ rustup default nightly
$ git clone https://github.com/radareorg/radeco
$ cd radeco
$ cargo install
```

#### Decompilation
```shell
$ echo '#include<stdio.h>\nint main() {printf("Hello, world.\\n"); return 0;}' | gcc -xc -
$ radeco
>> load a.out
Cannot find function here
[*] Fixing Callee Information
>> fn_list
sym._init
sym.imp.puts
entry0
sym.deregister_tm_clones
sym.register_tm_clones
sym.__do_global_dtors_aux
entry1.init
sym.main
sym.__libc_csu_init
sym.__libc_csu_fini
sym._fini
>> analyze sym.main
[+] Analyzing: sym.main @ 0x1139
  [*] Eliminating Dead Code
  [*] Propagating Constants
  [*] Eliminating More DeadCode
  [*] Eliminating Common SubExpressions
  [*] Verifying SSA's Validity
>> decompile sym.main
fn sym.main () {
    unsigned int tmp;
    *((rsp - 8)) = rbp
    tmp = sym.imp.puts("Hello, world.", rsi, rdx, rcx, r8, r9)
}
>>
```
Radeco can also connect to existing radare2 process.

Radare2 side
```shell
$ r2 a.out
[0x00001040]> aaa
[x] Analyze all flags starting with sym. and entry0 (aa)
[x] Analyze function calls (aac)
[x] Analyze len bytes of instructions for references (aar)
[x] Constructing a function name for fcn.* and sym.func.* functions (aan)
[x] Type matching analysis for all functions (afta)
[x] Emulate code to find computed references (aae)
[x] Analyze consecutive function (aat)
[0x00001040]> afn write_str @ sym.imp.puts
[0x00001040]> =h& 9090
Background http server started.
WARNING: Background webserver requires http.sandbox=false to run properly
[0x00001040]>
Starting http server...
open http://localhost:9090/
r2 -C http://localhost:9090/cmd/
[0x00001040]>
```

Radeco side
```shell
$ radeco
>> connect http://localhost:9090
[*] Fixing Callee Information
>> decompile sym.main
fn sym.main () {
    unsigned int tmp;
    *((rsp - 8)) = rbp
    tmp = write_str("Hello, world.", rsi, rdx, rcx, r8, r9)
    return
}
```
You can see radeco uses the information modified by radare2.

### Example

Original source code.
```c
void write_strlen(const char *str) {
    const char *iter = str;
    while (*iter) iter++;
    printf("%d\n", iter - str);
}
```

Disassembly of the compiled code
```
|   sym.write_strlen (int arg1);
|           ; var int local_18h @ rbp-0x18
|           ; var int local_8h @ rbp-0x8
|           ; CALL XREF from sym.main (0x1189)
|           0x00001139      55             push rbp
|           0x0000113a      4889e5         mov rbp, rsp
|           0x0000113d      4883ec20       sub rsp, 0x20
|           0x00001141      48897de8       mov qword [local_18h], rdi  ; arg1
|           0x00001145      488b45e8       mov rax, qword [local_18h]
|           0x00001149      488945f8       mov qword [local_8h], rax
|       ,=< 0x0000114d      eb05           jmp 0x1154
|       |   ; CODE XREF from sym.write_strlen (0x115d)
|      .--> 0x0000114f      488345f801     add qword [local_8h], 1
|      :|   ; CODE XREF from sym.write_strlen (0x114d)
|      :`-> 0x00001154      488b45f8       mov rax, qword [local_8h]
|      :    0x00001158      0fb600         movzx eax, byte [rax]
|      :    0x0000115b      84c0           test al, al
|      `==< 0x0000115d      75f0           jne 0x114f
|           0x0000115f      488b45f8       mov rax, qword [local_8h]
|           0x00001163      482b45e8       sub rax, qword [local_18h]
|           0x00001167      4889c6         mov rsi, rax
|           0x0000116a      488d3d930e00.  lea rdi, [0x00002004]       ; "%d\n" ; const char *format
|           0x00001171      b800000000     mov eax, 0
|           0x00001176      e8b5feffff     call sym.imp.printf         ; int printf(const char *format)
|           0x0000117b      90             nop
|           0x0000117c      c9             leave
\           0x0000117d      c3             ret
```

Decompiled code
```c
fn sym.write_strlen () {
    int local_18h;
    unsigned int tmp;
    int local_8h;
    *((rsp - 8)) = rbp
    local_18h = rdi
    local_8h = local_18h
    while (1) {
        if (!((1 ^ (((((((*(local_8h) as 64) | (local_8h & 18446744069414584320)) & 4294967295) as 8) & ((((*(local_8h) as 64) | (local_8h & 18446744069414584320)) & 4294967295) as 8)) - 0) & 255)) as 1)) {
            tmp = sym.imp.printf(8196, unknown, rdx, rcx, r8, r9)
            return
        } else {
            local_8h = (local_8h + 1)
        }
    }
}
```

## Overview

### Radeco Architecture

![diagram radare and radeco](/blog/images/radeco_architecture.png)

Radeco takes information from radare2 through r2pipe interface, then it converts assembly code to internal representation. After some analyse steps (e.g., deadcode elimination, constant propagation, common subexpressions elimination), the C-like expressions are recovered from it, and pseudocode is generated after graph modified to remove goto statements.

My work was to implement recovering expressions and C pseudocode generation from C-like CFG.

### Recovering expressions
This part of code located in `backend::lang_c::c_cfg_builder`. It recovers statements (assignment, if, goto, function call) and expressions from IR, which is assembly-like representation. Then we convert it to C-like representation with goto statements and gotos are structured into `while`/`do-while` statements with `No More Gotos` algorithm implemented by HMPerson1.

### Generating pseudo C-code from C-like CFG
The code located in `backend::lang_c::c_ast` and `backend::lang_c::c_cfg`. It converts C-like CFG to C-AST with goto statements. Currently, this code used to check whether recovering expressions correct or not, since `backend::ctrl_flow_struct` does the task.

## Further Implementaions
- Simplify the conditions in "IF" statements
- Recover return value and its type of decompiled function
- Write more generic interface so that it would be more extensible
- Improve r2 integration

## Other
- [radeco](https://github.com/radareorg/radeco) decompiler interface
- [Integration with r2](https://github.com/radareorg/radeco/pull/23)

## Links
- [GSoC proposal](https://drive.google.com/open?id=1u2LPw6GPMJOrPI0tyNrfoovyJif2Evc2fcFzMFsZ0fg)
- [Pull Requests](https://github.com/radareorg/radeco-lib/pulls?utf8=%E2%9C%93&q=is%3Apr+author%3Akriw)
- [Commits](https://github.com/radareorg/radeco-lib/commits?author=kriw&since=2018-05-01&until=2018-08-14)

