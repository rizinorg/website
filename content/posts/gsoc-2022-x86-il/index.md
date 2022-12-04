---
author: "DMaroo"
title: "GSoC 2022 - x86 ISA lifting for RzIL"
date: "2022-12-04"
summary: "A summary of all the work done on RzIL lifting for x86 ISA, and how you can use it."
tags: ["rizin", "gsoc"]
ShowToc: false
weight: 2
---

Hi! I'm [DMaroo](https://github.com/DMaroo), a GSoC 2022 mentee, working on IL migration for x86 ISA in Rizin. For the past few months, I have worked on implementing x86 instructions (from 8086, 80186, 80286 and 80386 instruction sets) in Rizin's intermediate language, [RzIL](https://github.com/rizinorg/rizin/blob/dev/doc/rzil.md).

The following article covers all the work, design decisions, challenges and future plans of the work that I've been doing. The RzIL can be accessed using `aez` commands (RzIL emulation).

## Relevant pull requests

Implementation of RzIL lifting for x86 instructions: [rizinorg/rizin#2747](https://github.com/rizinorg/rizin/pull/2747)
Adding tracing for x86 emulation in BAP's QEMU: [BinaryAnalysisPlatform/qemu](https://github.com/BinaryAnalysisPlatform/qemu/pull/21)

# Abstract

[RzIL](https://github.com/rizinorg/rizin/blob/dev/doc/rzil.md) is Rizin's intermediate language. It is designed for improved analysis of binaries by serving as an intermediate language to run the analysis loop on. This removes the need to write architecture-specific analysis code. Having an intermediate language also allows for many other features like taint analysis, symbolic execution and de-obfuscation.

My goal for the project was to "lift" the x86 architecture to RzIL. Lifting here means implementing the x86 instructions from the x86 ISA using opcodes present in the IL. The IL is largely based on [BAP's Core Theory](https://binaryanalysisplatform.github.io/bap/api/master/bap-core-theory/Bap_core_theory/Theory/index.html).

Throughout my GSoC period, I have lifted x86 instructions for majority of the instructions belonging in the 8086, 80186, 80286 and 80386 instruction sets. These include almost all the commonly used instructions one can find in modern x86 binaries, and hence would be enough to do fairly satisfactory analysis. I outline my work in detail below.

# Work

## Getting instructions from Capstone

Rizin uses [Capstone](https://www.capstone-engine.org/) for the disassembly of some of the instruction sets, including x86. So I had to start off by figuring out all the information that Capstone gives about an instruction. Capstone returns a `cs_x86` struct which contains the instruction bytes, operands, prefixes and other data. I wrapped it in a `X86ILIns` struct for convenience. Once I had the wrappers ready, I had to write general helper functions for generating the opcodes for variety of tasks like for getting and setting the operands, registers, memory, flags, and arithmetic overflow and so on.

## Setting up the helper functions

I have conceptually outlined the rough thought process for the IL lifting in one of the posts on [my blog](https://dmaroo.github.io/) about [SuperH IL lifting](https://dmaroo.github.io/superhlifting). Following a similar design process, I set up the functions to get and set the various entities involved. I also set up th functions for overflow/underflow and carry/borrow. More about the challenges faced during setting up these functions in the [challenges](#challenges) section. Once these were setup, the lifting was just reading the implementations off of the ISA manual and translating them to the IL.

## Implementing the instructions

This is probably the main part of the whole project. Now, I was supposed to go through the instruction manual's implementation for all the instructions and convert them to the IL. I could not however just blindly copy them, since the x86 instruction set frequently contains instructions with very weird special cases. Also, the x86 architecture is not simple, and contains multiple modes of operation, and also has segmentation and paging support. All of this makes it non-trivial to implement the instructions. Many of the instructions are not practically possible to implement given the current scope of the IL and the disassembler.

However, I did implement majority of the instructions in the 8086, 80186, 80286 and 80486 instruction set. That constitutes the meat of all the instructions used in x86 binaries. There are much more very specific and exotic instructions, but their relevance is diminishingly low. A sample implementation looks as follows:

```c
/**
 * ADD  dest, src
 * (ADD family of instructions)
 * Add
 * dest = dest + src
 * Possible encodings:
 *  - I
 *  - MI
 *  - MR
 *  - RM
 */
IL_LIFTER(add) {
	RzILOpEffect *op1 = SETL("op1", x86_il_get_op(0));
	RzILOpEffect *op2 = SETL("op2", x86_il_get_op(1));
	RzILOpEffect *sum = SETL("sum", ADD(VARL("op1"), VARL("op2")));

	RzILOpEffect *set_dest = x86_il_set_op(0, VARL("sum"));
	RzILOpEffect *set_res_flags = x86_il_set_result_flags(VARL("sum"));
	RzILOpEffect *set_arith_flags = x86_il_set_arithmetic_flags(VARL("sum"), VARL("op1"), VARL("op2"), true);

	return SEQ6(op1, op2, sum, set_dest, set_res_flags, set_arith_flags);
}
```

And the generated opcode looks something like this:

```
add byte [eax], al

(seq (set op1 (loadw 0 8 (+ (var eax) (bv 32 0x0)))) (set op2 (cast 8 false (var eax))) (set sum (+ (var op1) (var op2))) (storew 0 (+ (var eax) (bv 32 0x0)) (var sum)) (set _result (var sum)) (set _popcnt (bv 8 0x0)) (set _val (cast 8 false (var _result))) (repeat (is_zero (var _val)) (seq (set _popcnt (+ (var _popcnt) (ite (lsb (var _val)) (bv 8 0x1) (bv 8 0x0)))) (set _val (>> (var _val) (bv 8 0x1) false)))) (set pf (is_zero (smod (var _popcnt) (bv 8 0x2)))) (set zf (is_zero (var _result))) (set sf (msb (var _result))) (set _result (var sum)) (set _x (var op1)) (set _y (var op2)) (set cf (|| (|| (&& (msb (var _x)) (msb (var _y))) (&& (! (msb (var _result))) (msb (var _y)))) (&& (msb (var _x)) (! (msb (var _result)))))) (set of (|| (&& (&& (! (msb (var _result))) (msb (var _x))) (msb (var _y))) (&& (&& (msb (var _result)) (! (msb (var _x)))) (! (msb (var _y)))))) (set af (|| (|| (&& (msb (cast 4 false (var _x))) (msb (cast 4 false (var _y)))) (&& (! (msb (cast 4 false (var _result)))) (msb (cast 4 false (var _y))))) (&& (msb (cast 4 false (var _x))) (! (msb (cast 4 false (var _result))))))))
```

As you can see, the IL is quite large even for a simple instruction like `ADD`. This is because of all the extra work which needs to be done other than just adding the operands. The operand needs to be loaded from the proper memory location (which requires using the correct segment base register, correct scale and correct offset). Once the addition is done, the flags need to be set. Setting the flags is not a simple operation since it requires finding the parity bit (which requires XORing the bits in a loop). Once all the flag bits are set, the result is written back to the memory.

As of this post, 100+ such instructions have been lifted to the IL. There are more instructions to be lifted as well, but as I stated above, these are enough for a start and testing.

Once I was done with implementing the instructions, I added IL tests for the implemented instructions for the tests in `tests/db/asm/x86_*`. This part ensures that the generated IL code for the instructions is type-safe and doesn't have any memory issues.

## Enabling tracing in QEMU

Now to verify the semantics of the IL instructions, we use traces generated by QEMU and compare them with the effects of teh IL when executed by the RzIL VM. We use a [fork of QEMU](https://github.com/BinaryAnalysisPlatform/qemu), so that we can add the tracing code and modify QEMU source if needed. To add the tracing code, I had to familiarize myself with QEMU's code, and specifically the [TCG (Tiny Code Generator)](https://wiki.qemu.org/Documentation/TCG). Currently, adding the tracing has been almost done. However, there are just some very minor issues which need to be resolved.

<div id="challenges"></div>

# Challenges

<sup>This is more of an interesting-stuff kinda section ;)</sup>

## Registers

Choosing the correct registers without a chain of if-else statements or switch-case statements was one of the initial issues I faced. Not only do I have to resolve to the correct register, I should also be able to store and load from the same global IL variable when I reference different sections of a register (i.e. `AL`, `AX`, `EAX` and `RAX` overlap over the same global IL variable). Along with this, I should also have the ability to choose the largest register of a certain bitness (for example, `RAX` for 64-bit, `EAX` for 32-bit and `AX` for 16-bit) without using switch-case statements. I decided to do this using a statically defined array of registers and their get and set functions (`gpr_lookup_table`, array of `struct gpr_lookup_helper_t`). This lead to no sort of if-else or switch-case constructs, and I could reuse the same functions for getting and setting the same sections o a register.

## Operands

Also, there had to be a general interface for accessing the operands, irrespective of whether they were a register, or a memory location, or an immediate value. Writing the helper functions is easy, but providing ease of usage and removing duplication code requires thought. I have also slightly touched upon this in the [SuperH IL lifting post](https://dmaroo.github.io/superhlifting) on [my blog](https://dmaroo.github.io/).

## IL implementation

The descriptions in the x86 manual for some instructions get pretty complex and intense. Making sure that they are implemented correctly is not easy. This is exactly why we have tracetesting to verify the semantics, but manual verification never hurts. Also, the IL gets pretty large for many instructions, and to avoid this, it is important to choose a semantically correct, yet concise implementation. This involves removing unnecessary casting, using loops, removing duplicating and so on. In fact, removing duplication by using variables brings down the IL code size by a lot.

## QEMU

QEMU's codebase is a great for learning purposes. It consists of complex C constructs (tons of non-trivial preprocessor macros), but at the same time it is quite well-written, and also reasonably well documented. However, debugging the codebase is not that easy, since the code generates a buffer (`code_gen_buffer`), which then executes more code.

# Future Work

The work I have done in my duration is just the core part of the lifting. The lifting is far from being stable and ready-to-use. It is more of an pre-alpha release than a finished feature. The next steps will be as follows.

 - **Tracetest**: Thoroughly tracetest the implementations to make sure the semantics are right. 
 - **API**: Add Rizin commands and API, which would be wrappers around the IL, so as to visualize and access the IL. A similar graphical widget should also be added to Cutter.
 - **Documentation**: Document the IL and its usage in the Rizin book.
 - **Analysis**: Integrate the IL in the analysis loop to lead to a better analysis.
 - **Instructions**: Add lifting for more x86 instructions.

The last two options haven't been thoroughly thought out by me yet, but they do seem to be reasonable future goals.

# Closure

I would say that I had a very educative experience throughout the period. I learnt quite a lot about the x86 architecture and the instruction set in the past few months. I was not able to meet all my goals for my GSoC project, but I think the work done until now is a good enough checkpoint and I plan to continue the work even after my GSoC tenure ends.

I would like to thank my mentors, [Anton Kochkov](https://github.com/XVilka), [Deroad](https://github.com/wargio) and [Florian Märkl](https://github.com/thestr4ng3r), for all the guidance they have provided me throughout my project. I look forward to keep working with them :)
