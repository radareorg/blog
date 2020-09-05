+++
date = "2020-09-06T00:00:00+00:00"
aliases = [ "SLEIGH Disassembler Backend" ]
draft = false
slug = "sleigh_disassembler_backend"
title = "GSoC 2020: SLEIGH Disassembler Backend"
+++

## Introduction

Hello all, I'm [Jiaxiang Zhou](https://github.com/FXTi) from China. I was lucky to be selected as a participant of Radare2 project this year. My main work was to  integrate SLEIGH as a disassembly backend into Radare2. [r2ghidra-dec](https://github.com/radareorg/r2ghidra-dec) was my main working repository, aiming to delivering Ghidra's decompiler to Radare2. It could be renamed as `r2ghidra` since it would become not only a decompiler but a complete bridge between Radare2 and Ghidra after this project.

Special thanks should be given to my mentors, [Florian Märkl](https://github.com/thestr4ng3r), [Giovanni](https://github.com/wargio), and [Anton](https://github.com/XVilka). Your patience and guidance are all well appreciated. I couldn't have completed this project without your support.

Here's the [slides](https://blog.fxti.xyz/2020/08/31/GSoC2020-SLEIGH-Plugin/r2con2020-SDB.pdf) made for r2con2020.


## RAsm plugin

SLEIGH disassembler has been deeply embedded into C++ codebase of decompiler. So the solution is clear:

### Isolate SLEIGH from C++ codebase of Ghidra's decompiler

To get full access of `Sleigh` and low level spec file and interfaces, I implemented a class(`SleighAsm`) just like lite version of `Architecture` . This class will export P-codes and registers' info parsed from spec file. It enable us to disassemble all valid instructions on demand:

![](https://blog.fxti.xyz/2020/08/31/GSoC2020-SLEIGH-Plugin/asm_ghidra.png)


## RAnal plugin

SLEIGH will give out P-codes as IR to describe what instruction does. So I had to analysis on P-codes to extract control flow, type info. When it came to ESIL, things got more tricky because P-code's model and ESIL are different. What's more, P-codes support float number operation, which ESIL doesn't.

### Port SleighInstructionPrototype

Ghidra's C++ codebase concentrate on decompiler, so it focus on function-level analysis. There's classes like `Funcdata` to analysis intra-function flow. But instruction-level analysis tool only exists in JAVA codebase. So I had to port `SleighInstructionPrototype` from JAVA to C++. This enables control flow info extraction on `Constructors` (lower than P-code). This port work was tough to minimize the changes on original Ghidra codebase. And I eventually managed to port whole `SleighInstructionPrototype` with only two private fields exported!

```c++
diff --git a/Ghidra/Features/Decompiler/src/decompile/cpp/context.hh b/Ghidra/Features/Decompiler/src/decompile/cpp/context.hh
index 3372e5ac5..5d2e108f7 100644
--- a/Ghidra/Features/Decompiler/src/decompile/cpp/context.hh
+++ b/Ghidra/Features/Decompiler/src/decompile/cpp/context.hh
@@ -89,6 +89,8 @@ private:
   ConstructState *base_state;
   int4 alloc;                  // Number of ConstructState's allocated
   int4 delayslot;              // delayslot depth
+protected:
+  ConstructState **getBaseState(void) { return &base_state; }
 public:
   ParserContext(ContextCache *ccache);
   ~ParserContext(void) { if (context != (uintm *)0) delete [] context; }
diff --git a/Ghidra/Features/Decompiler/src/decompile/cpp/sleigh.hh b/Ghidra/Features/Decompiler/src/decompile/cpp/sleigh.hh
index 22b9c7a21..5e6617db8 100644
--- a/Ghidra/Features/Decompiler/src/decompile/cpp/sleigh.hh
+++ b/Ghidra/Features/Decompiler/src/decompile/cpp/sleigh.hh
@@ -113,6 +113,7 @@ protected:
   ParserContext *obtainContext(const Address &addr,int4 state) const;
   void resolve(ParserContext &pos) const;
   void resolveHandles(ParserContext &pos) const;
+  ContextCache *getContextCache(void) { return cache; }
 public:
   Sleigh(LoadImage *ld,ContextDatabase *c_db);
   virtual ~Sleigh(void);
```

And this is control flow extracted only from P-code results:

![](https://blog.fxti.xyz/2020/08/31/GSoC2020-SLEIGH-Plugin/control_flow.jpg)

### P-code to ESIL translation

Ghidra's emulate system based on P-code is quite different from ESIL. First, it's not stack-based, and it can randomly access to any middle variables.

Here's my design:

1. Add `PICK` to take values from stack to stack top

   I will leave middle variables intentionally on stack and retrieve them when this middle variable is needed.

2. Add `GET` to retrieve register's value to stack

   Sometimes register's name is just one char, ESIL will confuse if it's compared with a number. Most importantly, I need `GET` to retrieve float number to stack without changing ESIL's original codes in Radare2.

3. Override `=` to store value from stack back to register

   This is to pair with `GET` for float number handling. You will notice that all element except register(used as destination) are immediate value.

4. Override `[4]` and `[8]` to read float number from memory

   When a float number is written into a register/memory location, I will record its register name/memory address to track until it's overwritten by anything except float number.

5. Override `=[4]` and `=[8]` to store float number to memory

   As above.

6. Add serials of float operation.

When real translation is running, plugin will employ a stack to emulate middle variables left on stack. This will help calculate offset of middle variables(unique varnodedata) used as argument of current P-code.

Here's example:

![](https://blog.fxti.xyz/2020/08/31/GSoC2020-SLEIGH-Plugin/esil.png)

### Pattern match on P-code to analysis type of instruction

SLEIGH only provide P-codes. But P-codes doesn't tell what the instruction is. And the multi-arch support of SLEIGH make thing even more complex.

I made an [overview](https://github.com/radareorg/r2ghidra-dec/pull/120#issuecomment-673083656) on all `R_ANAL_OP_TYPE_*` and summary their patterns. Hope to do the pattern match based on P-codes and know what type the instruction is. Sounds crazy, but I not only typed instructions successfully, I also managed to recover arguments of associated instructions!

Here's example:

![](https://blog.fxti.xyz/2020/08/31/GSoC2020-SLEIGH-Plugin/type.png)


## Summary

RAsm and RAnal plugins are both workable. Ready to provide information recovered from Ghidra's SLEIGH disassembler.

![](https://blog.fxti.xyz/2020/08/31/GSoC2020-SLEIGH-Plugin/anal.png)

![](https://blog.fxti.xyz/2020/08/31/GSoC2020-SLEIGH-Plugin/flow.png)

Related commits:

https://github.com/thestr4ng3r/ghidra/commit/50acb42da120072ce1c02a15004bc7e74a682599

https://github.com/thestr4ng3r/ghidra/commit/7bde3b54b43230601363f89b0214ab4bdba8bf6f

https://github.com/radareorg/r2ghidra-dec/commit/59473e5c53d8724cfee489968e0990a31396ef90

https://github.com/radareorg/r2ghidra/commit/e083b183c9954c899fd4f945547683689c9d00a4

