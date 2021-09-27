---
author: "xvilka"
title: "Rizin Summer of Code 2021 Summary"
date: "2021-09-27"
summary: ""
tags: ["rizin", "cutter", "rsoc"]
ShowToc: true
TocOpen: false
weight: 2
---

# Rizin Summer of Code 2021 Summary

[RSoC 2021](https://rizin.re/posts/rsoc-2021-announcement/) is [officially finished](https://keenlab.tencent.com/zh/2021/08/24/2021-rsoc-summary/) and we are happy to congratulate both participants for passing the program and completing the most important parts of their tasks.

# Basstorm: Types analysis

Hello, I am Basstorm. Over the past two months, I had a fulfilling summer as one of the participants of RSoC. The main subject of RSoC was to improve the `Type` module.

At first, I fixed several bugs in the new `tree-sitter` based type parser. The new type parser brings us the ability to parse a C type defined as a string. After that, I migrated the type constraints from RzAnalysis to the new RzType module, which makes the type constraints management easier.

```
[0x00000530]> e analysis.types.constraint=true
[0x00000530]> aaa
[x] Analyze all flags starting with sym. and entry0 (aa)
[x] Analyze function calls (aac)
[x] Analyze len bytes of instructions for references (aar)
[x] Check for classes
[x] Type matching analysis for all functions (aaft)
[x] Propagate noreturn information
[x] Use -AA or aaaa to perform additional experimental analysis.
[0x00000530]> s sym.range_small
[0x0000063a]> pdf
            ; CALL XREF from main @ 0x720
/ sym.range_small (int64_t arg1);
|           ; var int64_t var_14h  { > 0x0 && <= 0x9} @ rbp-0x14       ;constraint
|           ; var int64_t var_8h @ rbp-0x8
|           ; var int64_t var_4h  { } @ rbp-0x4
|           ; arg int64_t arg1 @ rdi
|           0x0000063a      push  rbp
|           0x0000063b      mov   rbp, rsp
|           0x0000063e      sub   rsp, 0x20
|           0x00000642      mov   dword [var_14h], edi                 ; arg1
```

For historical reasons, Rizin has never had support for global variables, which means we can't identify and set a certain global variable, which is detrimental to our analysis. I have added support for global variables so that we can easily manipulate a global variable from the command line.

```
[0x00000000]> avg?
Usage: avg[jadmnt]   # Global variables
| avg[j] [<var_name>]             # show global variables
| avga <var_name> <addr> <type>   # add global variable manually
| avgd <addr>                     # delete the global variable at the addr
| avgm <name>                     # delete global variable with name
| avgn <old_var_name> <new_var_name> # rename the global variable
| avgt <var_name> <type>          # change the global variable type
[0x00000000]> avga foo 0x100 char
[0x00000000]> avg
global char foo @ 0x100
[0x00000000]> avgt foo int
[0x00000000]> avg
global int foo @ 0x100
[0x00000000]>
```

In addition, I completely refactored [PDB Parser](https://github.com/rizinorg/rizin/pull/1517) to make it better cross-platform. Previously, PDB Parser had a lot of problems with its functionality, such as missing information, parsing errors, and unused types. All these problems are solved in this refactoring.

```
$ rizin Project1.exe
 -- Use scr.accel to browse the file faster!
[0x00401703]> idpi ./Project1.pdb
·········
struct std::_Char_traits<char32_t,unsigned int> {
        char32_t char_type;
        uint32_t int_type;
        int64_t off_type;
        char32_t copy(char32_t * arg0, const char32_t * arg1, const uint32_t arg2);
        char32_t _Copy_s(char32_t * arg0, const uint32_t arg1, const char32_t * arg2, const uint32_t arg3);
        char32_t move(char32_t * arg0, const char32_t * arg1, const uint32_t arg2);
        int32_t compare(const char32_t * arg0, const char32_t * arg1, uint32_t arg2);
        uint32_t length(const char32_t * arg0);
        const char32_t * find(const char32_t * arg0, uint32_t arg1, const char32_t * arg2);
        bool eq(const char32_t * arg0, const char32_t * arg1);
        bool lt(const char32_t * arg0, const char32_t * arg1);
        char32_t to_char_type(const uint32_t * arg0);
        uint32_t to_int_type(const char3_t * arg0);
        bool eq_int_type(const uint32_t * arg0, const uint32_t * arg1);
        uint32_t not_eof(const uint32_t * arg0);
        uint32_t eof();
 }
·········
[0x00401703]> tuc
·········
union __m64 {
        uint64_t m64_u64;
        float m64_f32[8];
        unsigned char m64_i8[8];
        int16_t m64_i16[8];
        int32_t m64_i32[8];
        int64_t m64_i64;
        unsigned char m64_u8[8];
        uint16_t m64_u16[8];
        uint32_t m64_u32[8];
};
·········
```
Currently, Heersin has completed the new [RzIL](https://github.com/rizinorg/rizin/pull/1663), but it still lacks support for many architectures. So I am now [porting the 8051 architecture](https://github.com/rizinorg/rizin/pull/1609) from the old ESIL to the new RzIL, and I will be working with Heersin to port more architectures to the new IL afterwards.

During this RSoC, I grew a lot and learned a lot of development skills that I would not normally be exposed to. I would like to especially thank my mentor Anton Kochkov for his selfless help. I would also like to thank all the community members for their help!

# Heersin: New Rizin IL
Hi, I'm Heersin, I particpated in RSoC this summer to introduce a new Intermediate Language and refactor ESIL related code. Rizin previously used ESIL([a stack-based IL](https://book.rizin.re/disassembling/esil.html)) as its IL to analyse binary. In fact, ESIL is neither user friendly nor developer friendly, those are some of the reasons that led to this work. We take [BAP's Core Theory](http://binaryanalysisplatform.github.io/bap/api/odoc/bap-core-theory/Bap_core_theory/index.html) as our new IL. Because it's designed to be similar to SMT, and it may be the latest IL (the most "fashionable" one) we can trust for now.

In the first few days, I didn't have any clue about implementing a Core Theory VM, so I started to work on the some basic data structures (**Bool**/**BitVector**/**Array**) used in VM. They are basic types in core theory, we can emulate other types (`ut8`/`ut16`/`ut32`/`ut64`) by using bitvector and bool.

After that, I focused on the concepts in VM and the execution procedure. In short, there are **Variable** and **Value** in the Core Theory VM. A **Variable** is a symbol while a **Value** represents the evaluating result of an expression. `read register` is used to get the value of a variable and `write register` is used to assign a value to a variable. **Memory** is a Hashtable (kv-map), where the address is the key and the data is the value. The **Memory** concept is similar to the SMT Arrays theory where both values and indexes are **Bitvectors**.

Then, I uplifted the brainfuck to test the new IL. That's the uplifted expression.
```
# print mode
# ++++++++++[>+++++++>++++++++++>+++>+<<<<-]>++.>+.+++++++..+++.>++.<<+++++++++++++++.>.+++.------.--------.>+.>.
(STORE (VAR ptr) (ADD (LOAD (VAR ptr)) (INT 1)))
(STORE (VAR ptr) (ADD (LOAD (VAR ptr)) (INT 1)))
(STORE (VAR ptr) (ADD (LOAD (VAR ptr)) (INT 1)))
(STORE (VAR ptr) (ADD (LOAD (VAR ptr)) (INT 1)))
(STORE (VAR ptr) (ADD (LOAD (VAR ptr)) (INT 1)))
(STORE (VAR ptr) (ADD (LOAD (VAR ptr)) (INT 1)))
(STORE (VAR ptr) (ADD (LOAD (VAR ptr)) (INT 1)))
(STORE (VAR ptr) (ADD (LOAD (VAR ptr)) (INT 1)))
(STORE (VAR ptr) (ADD (LOAD (VAR ptr)) (INT 1)))
(STORE (VAR ptr) (ADD (LOAD (VAR ptr)) (INT 1)))
(BRANCH (LOAD (VAR ptr)) <NOP> (GOTO ]0))
(SET ptr (ADD (VAR ptr) (INT 1)))
(STORE (VAR ptr) (ADD (LOAD (VAR ptr)) (INT 1)))
...
(SET ptr (SUB (VAR ptr) (INT 1)))
(STORE (VAR ptr) (SUB (LOAD (VAR ptr)) (INT 1)))
(BRANCH (INV (LOAD (VAR ptr))) <NOP> (GOTO [0))
(SET ptr (ADD (VAR ptr) (INT 1)))
(STORE (VAR ptr) (ADD (LOAD (VAR ptr)) (INT 1)))
(STORE (VAR ptr) (ADD (LOAD (VAR ptr)) (INT 1)))
(GOTO write)
(SET ptr (ADD (VAR ptr) (INT 1)))
(STORE (VAR ptr) (ADD (LOAD (VAR ptr)) (INT 1)))
(GOTO write)
(STORE (VAR ptr) (ADD (LOAD (VAR ptr)) (INT 1)))
...
(GOTO write)
(SET ptr (ADD (VAR ptr) (INT 1)))
(STORE (VAR ptr) (ADD (LOAD (VAR ptr)) (INT 1)))
(GOTO write)
(SET ptr (ADD (VAR ptr) (INT 1)))
(GOTO write)
```

Next step is to integrate new IL with Rizin, and porting `analysis_bf` to use it. The analysis code is huge and ESIL is tightly integrated within it. I removed some dead code and reorganized the directory structure with the help from community members. Moreover, I added new structures for the `trace` and `stat` (they are used to collect info about reg/mem read/write) to replace the `sdb` approach with vectors and make it easier to understand. Then I continued to integrate the new IL; added `aezi` and `aezs` commands for `init VM` and `step emulate` respectively.

```
[0x00000000]> aezi
[0x00000000]> aezs 390
Hello World!
[0x00000000]> aezi
```

Porting more architectures will be a huge work. I will continue to contribute to rizin and improve the IL part.

I am grateful for such an opportunity to participate in RSoC and contribute to Rizin. There is a friendly atmosphere and I learned a lot. I want to give special thanks to my mentor XVilka for his guidance and help, and also to ret2libc, Ivg, Wargio, Pelijah, Thestr4ng3r and 08A for answering my questions and giving feedback on my PRs.
