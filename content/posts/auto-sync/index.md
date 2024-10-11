---
author: "Rot127"
title: "Auto-Sync - Generating disassembler plugins"
date: "2024-01-27"
summary: "Auto-Sync"
tags: ["dev", "rizin", "capstone"]
ShowToc: false
weight: 2
---

_Updated on 2024.10.01_

A disassembler is obviously a must-have tool to do any reversing task.
But using just any disassembler, especially for frameworks like Rizin, doesn't really do it.

There are several capabilities which would be nice to have.

It should:

- Be correct. And if it isn't, it should be easy to test and spot the error (in our case we want to compare the output directly to `llvm-objdump`).
- Provide a single API for multiple architectures.
- Support niche architectures or make it relatively easy to add them.
- Apart from the text disassembly, provide additional information about the operands and other meta-data.
- Be easy to update when new processor extensions come out.
- Relatively lightweight.
- Written in C or any other language that is easy to integrate into C/C++ software (specifically needed by Rizin/Cutter)

One of the first disassembler engines which was capable of some of those points was Capstone.
Quynh Nguyen Anh, the author of Capstone, figured that all the information we need, is basically
already there in compiler projects like LLVM.

Obviously, for compilation you need the same and much more information you would need for disassembling.
And you need them in a well-defined and machine-readable way.

So, what Capstone did, was re-implementing the LLVM disassembler logic in C,
add meta-data for each instruction from the architecture definitions (also given in the LLVM-project)
and add a single API to interact with it.

To summarize, Capstone is in the end:
- A more lightweight API than LLVM because it re-implements only the necessary code for disassembly from LLVM.
- Can support as many architectures as LLVM supports (if someone ports them to Capstone).
- Provides more information than the textual disassembly one gets via `llvm-objdump`.
- Relies on a well maintained and large project, which will (likely) be there even in 10+ years and is managed by people who know more about the architectures.

The big problem with Capstone was though, that it hadn't a working update mechanism.
There were a bunch of Python scripts and very little documentation. Definitely an unsustainable solution.

Due to this, Capstone became outdated over the years and most disassembler modules
didn't support modern processor extensions.

## What can be done?

Besides LLVM we attempted once to generate a disassembler module for the Hexagon architecture (a DSP architecture from Qualcomm). But instead of LLVM, we used the ISA PDF for our first try.
We parsed it and generated the decoding tables for the instructions.
This worked, but was a little messy. Parsing PDF files is not fun and as soon as the PDF file changes somehow, stuff is broken again.
Also, it is hard to test if you actually extracted the encoding information from the PDF correctly.

Our second attempt uses LLVM. LLVM provides a way to get the definitions of an architecture in JSON format (`llvm-tblgen --dump-json`). The JSON dump has all the details about instructions you can wish for.
Opcodes, operand types, read/write info and more. Pretty much anything you could wish for.
With this experience we decided that LLVM proved to be a good source for disassembler generation.

Now, with this experience we decided we could extend Capstone with a proper updater.
The alternative, implementing something Capstone like from scratch in a new project, did not really seem a good idea.
Capstone has already a large user base, and we would need to migrate to the new tool as well.
The last point is maybe annoying but doable. Getting a user base again is a way harder task.

So over the last two years we added an updater to Capstone.
With it, we updated some core modules (`ARM`, `AArch64`, `PPC` + `Paired Single`, `SystemZ`, `Mips` + `NanoMips`) and added new ones (`Alpha`, `TriCore`, `Xtensa`, `HPPA`).
And to our delight [jiegec](https://github.com/jiegec) and [FurryAcetylCoA](https://github.com/FurryAcetylCoA) added support for LoongArch.

In the following blog post we'll not just explain the update procedure in detail,
but also reflect on some challenges and problems you run into when you generate
disassemblers.

# How LLVM generates its disassemblers

[LLVM](https://github.com/llvm/llvm-project/tree/main/) is our ground truth Capstone is built on. So let's start with it.

**How is the LLVM disassembler generated and how does it work?**

LLVM defines its various supported architectures in a language, specifically designed for this purpose.
Each target's instructions, instruction operands, scheduling information and more is written in the [TableGen](https://llvm.org/docs/TableGen/ProgRef.html) language.
The definitions can be found in `llvm/lib/Target/<TARGET-NAME>/*.td`.

> Please note, that from now on we use "target" and "architecture" are interchangeable terms.
> In the LLVM realm we speak about a target. In Capstone context about an architecture.

Since each target is defined in the same way, LLVM can apply the same procedures on them to generate C++ code with it.
This is way better than implementing every target directly in C++.
Because otherwise a disassembler must be implemented again and again for each target. This would be of course too much effort.
With TableGen every instruction is already well-defined in the `td` files. So, LLVM uses a universal method to generate a decoder for each of them.
There is still some manual work left. E.g. handling edge cases and parsing operand bits. But the core logic is generated.

To use the content of the `td` files in a programmable way, the `llvm-tblegen` tool parses them and converts its content into C++ classes and saves them in a `RecordKeeper`.
The `RecordKeeper` class is TableGen's internal representation of the `td` files content. Which can now be used to generate arbitrary code.
These classes basically hold all the `td` file information in a uniform and programmable way.
The C++ classes belong logically to the so called `CodeGen` layer. Which, as the name already says, are used to generate code.

Example:

Definition of the ARM `setend` instruction in TableGen:
```python
def SETEND : AXI<(outs), (ins setend_op:$end), MiscFrm, NoItinerary,
                 "setend\t$end", []>, Requires<[IsARM]>, Deprecated<HasV8Ops> {
  bits<1> end;
  let Inst{31-10} = 0b1111000100000001000000;
  let Inst{9} = end;
  let Inst{8-0} = 0;
}
```

becomes a C++ class of the form of this

```cpp
SETEND {	// InstructionEncoding Instruction InstTemplate Encoding InstARM XI AXI Requires Deprecated
	// Instruction bits, the operand bits are marked (in this case only one bit for "end").
  field bits<32> Inst = { 1, 1, 1, 1, 0, 0, 0, 1, 0, 0, 0, 0, 0, 0, 0, 1, 0, 0, 0, 0, 0, 0, end{0}, 0, 0, 0, 0, 0, 0, 0, 0, 0 };
  field bits<32> Unpredictable = { 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0 };
  field bits<32> SoftFail = { 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0 };
  int Size = 4;
  string DecoderNamespace = "ARM";
  list<Predicate> Predicates = [IsARM];
  string DecoderMethod = "";
  bit hasCompleteDecoder = 1;
  string Namespace = "ARM";
	// Note this list of in and out operands. Setend has only an operand which is read and no operands it writes.
  dag OutOperandList = (outs);
  dag InOperandList = (ins setend_op:$end);
  string AsmString = "setend	$end";
  list<dag> Pattern = [];
  list<Register> Uses = [];
  list<Register> Defs = [];
  int CodeSize = 0;
  int AddedComplexity = 0;
  bit isPreISelOpcode = 0;
  bit isReturn = 0;
  bit isBranch = 0;
	...
  bit isBarrier = 0;
  bit isCall = 0;
  bit isAdd = 0;
  bit isTrap = 0;
  bit canFoldAsLoad = 0;
  bit mayLoad = ?;
  bit mayStore = ?;
  bit mayRaiseFPException = 0;
	...
  bit doubleWidthResult = 0;
  SubtargetFeature DeprecatedFeatureMask = HasV8Ops;
  bits<1> end = { ? };
}
```

Note, that the C++ class above has the same structure for each target. Hence, LLVM's code generation, can reason on them without the need to know specific target details.

Now, what code is actually generated? This depends on what you need. TableGen has several backends.
Each of them uses the `RecordKeeper's` content to generate different files.
For example, the [RegisterInfo](https://github.com/llvm/llvm-project/blob/main/llvm/utils/TableGen/RegisterInfoEmitter.cpp) backend generates several tables with information about target registers.
As mentioned above, the `RegisterInfo` backend doesn't need to know details about targets specific registers.
It just implements methods to generate an enumeration with all register names. Or it generates tables which map registers to their alias, or lookup tables which map bits to a register ID.

Generating enumerations is nice, but more complex C++ code is, of course, also generated.
For us very relevant is the decoding logic, which decodes a byte sequence into a Machine Code instruction.
A Machine Code instruction (`MCInst`) is the class which represents a target's decoded instruction.
It holds the ID of the instruction, its operands, some flags (`isBranch` etc.), and some more.

Decoding procedures from bytes to `MCInst` are the same for each target (except x86, because _historical reasons_ I guess).
In the CodeGen layer we still know the encoding of each instruction of a target. Another backend, the `DecoderEmitter`, consumes these encodings and
builds a state machine over them. The generated state machine simply checks certain bits and transitions into states. The end state is either an identified instruction or the disassembly failed.
After the instruction ID is decoded, a big switch case is walked over to call the different decoder methods of the instruction's operands.

    Checkout [`PPCGenDisassemblerTables.inc`](https://github.com/capstone-engine/capstone/blob/next/arch/PowerPC/PPCGenDisassemblerTables.inc). It contains this state machine. Or see the examples below.

The key is: the state machine table and the big switch cases can be generated independently of the target.
Each target still needs to implement the operand decoders, because those are unique, but this is essentially it.
It saves quite some work, compared to implementing the decoding logic every time again and again.

**Excerpt from state machine**

```cpp
static const uint8_t DecoderTableARM32[] = {
//            What to do in the state | Bits to check or to extract
/* 0 */       MCD::OPC_ExtractField, 25, 3, // Extract 3 bits at offset 25 from byte sequence
/* 3 */       MCD::OPC_FilterValue, 0, 47, 14, 0 // Check the certain bits for properties and transition to another state depending on the result.
/* 8 */       MCD::OPC_ExtractField, 21, 1,
/* 11 */      MCD::OPC_FilterValue, 0, 110, 7, 0
/* 16 */      MCD::OPC_ExtractField, 24, 1,
/* 19 */      MCD::OPC_FilterValue, 0, 139, 1, 0
/* 24 */      MCD::OPC_ExtractField, 4, 1,
/* 27 */      MCD::OPC_FilterValue, 0, 123, 0, 0
/* 32 */      MCD::OPC_ExtractField, 22, 2,
/* 35 */      MCD::OPC_FilterValue, 0, 25, 0, 0
/* 40 */      MCD::OPC_CheckPredicate, 0, 11, 0, 0 // Check if a predicate is fulfilled (CPU feature X enabled etc.) and transision in a certain state depending on the result.
...
```

**Excerpt of the operand decoding switch statement**
```cpp
  switch (Idx) {
  default: llvm_unreachable("Invalid index!");
  case 0:
    // Extract bits of operand
    tmp = fieldFromInstruction(insn, 12, 4);
    // Decode a GPR register and check if it worked.
    if (!Check(S, DecodeGPRRegisterClass(MI, tmp, Address, Decoder))) { return MCDisassembler::Fail; }
    tmp = fieldFromInstruction(insn, 16, 4);
    if (!Check(S, DecodeGPRRegisterClass(MI, tmp, Address, Decoder))) { return MCDisassembler::Fail; }
    tmp = fieldFromInstruction(insn, 0, 4);
    if (!Check(S, DecodeGPRRegisterClass(MI, tmp, Address, Decoder))) { return MCDisassembler::Fail; }
    tmp = fieldFromInstruction(insn, 28, 4);
    if (!Check(S, DecodePredicateOperand(MI, tmp, Address, Decoder))) { return MCDisassembler::Fail; }
    tmp = fieldFromInstruction(insn, 20, 1);
    if (!Check(S, DecodeCCOutOperand(MI, tmp, Address, Decoder))) { return MCDisassembler::Fail; }
    return S;
```

A target's disassembler module in LLVM consists effectively of two parts. The generated logic (those are written to `.inc` files) and handwritten decoder and printing methods.
Decoders for operands like `DecodeGPRRegisterClass` from above, need to be implemented per target. They cannot be generated currently.

The handwritten code is in files like `<ARCH>Disassembler.cpp` or `<ARCH>AsmWriter.cpp` in their
respective target source directories.

## LLVM to Capstone

Capstone simply copies the LLVM disassembler and enriches the output.
Because we do not want to build LLVM just to build Capstone (LLVM is a huge dependency), we have to tackle two problems:

- LLVM code is in C++, Capstone in C.
- The LLVM disassembler has no knowledge about read/write access of operands or instruction groups. Theoretically it could, but it is not implemented. For Capstone we need this information though.

### C++ and C

For Capstone we need the C++ files in C. We also need the generated `*.inc` files, as well as handwritten disassembler components (`<ARCH>Disassembler.cpp` and `<ARCH>AsmWriter.cpp` from above).

Lets see how to get them in C.

### Generate C code with TableGen

We already described the generation procedure of the `.inc` files above in detail.
Though what we have not mentioned is the way the actual code is emitted. The problem with the TableGen backends is, they do not separate the generation of their data from the actual printing of the code.

For example, the backend which generates the [`AsmWriter`](https://github.com/capstone-engine/llvm-capstone/blob/auto-sync/llvm/utils/TableGen/AsmWriterEmitter.cpp) (the module which prints an asm string of an instruction) mixes it's table generation with emitting code.
There is no clear separation between generating abstract objects, like state machines and tables, and printing them into code. It is all intermingled.

This goes so far that it is even allowed to specify custom code in the `td` files for operands or instructions.

So, if we want TableGen backends emit C, we either need to redesign and rewrite them from scratch or patch them.
Designing it from scratch is a rather complex task. And needs a lot of thought ([see this discussion](https://reviews.llvm.org/D138323)).
Simply because of time constraints and because we don't know if it will be merged, we sided with patching.

Our [patched TableGen](https://github.com/capstone-engine/llvm-capstone) backends work pretty straight forward. We add two new classes which only emit code.
[`PrinterLLVM`](https://github.com/capstone-engine/llvm-capstone/blob/auto-sync/llvm/utils/TableGen/PrinterLLVM.cpp) and [`PrinterCapstone`](https://github.com/capstone-engine/llvm-capstone/blob/auto-sync/llvm/utils/TableGen/PrinterCapstone.cpp).
The `PrinterLLVM` emits the standard C++ code from LLVM. `PrinterCapstone` emits our C code.
Each backend gets one of those printer classes assigned. And whenever it emits code, it calls the corresponding method of the printer.
In practice, we simply moved the emitting code from the backend to the printer classes.

### General design problems with TableGen backends

The problem is, it is ugly. Although we are now able to emit C, it only works because C and C++ are so similar.
An array initialization in C++ is almost the same as in C. So the code structure is basically the same.
But there is no way to emit the same information (tables, functions etc.) in a different order or in a fundamentally different language (think of Lisp).
This is a simple necessity how the backends were build. Because there is no clear separation between generating logic and printing it as code,
the backends in the current form cannot be refactored nicely to emit code in other languages than C++.

Most of it is also untouched for 10 years and was never modernized. This is understandable, since it never really was necessary.
It works for the current use case (generate code for LLVM tools). But it doesn't allow using the generated logic in any other way.

For example, the state machine for decoding bytes to instructions is useful logic. Also for non-LLVM projects. It could be written once and used by everyone else.
But it is pretty much hard-coded to provide the state machine only in C++.

This is unfortunate. LLVM is a huge project, and many tools use the information about architectures it provides.
Providing these kinds of often used algorithms in an accessible way, would be a nice addition.

### Translating C++ to C

But back to the problem at hand. While we have now the generated C code, we still have handwritten code in C++.
As mentioned before, the operand decoder and printer methods are handwritten in LLVM. Additionally, some edge cases are handled there as well.
These files have to be translated from C++ to C.

Doing this by hand is a tedious task. We need to do it for every architecture module again and again. Because those files are not shared between targets.
And if we add a new architecture module from LLVM to Capstone, we would need to
translate multiple thousand lines of C++ to C.

This of cause is not particular fun and hinders people to do it at all.
Hence, we built the Auto-Sync framework to do most of the annoying work.

The translation process follows a simple procedure. We have a bunch of patches defined. Each patch replaces certain syntax in an C++ file with its C equivalent.

To find the patterns we want to replace we use [tree-sitter](https://tree-sitter.github.io/tree-sitter/). It allows us to query for specific syntax in the abstract syntax tree (AST) of the file.
And since we translate source code, it is way easier to search in an AST, instead in the file content itself.

To control the patching, we have a controller called [`CppTranslator`](https://github.com/capstone-engine/capstone/tree/next/suite/auto-sync/Updater/CppTranslator). It simply:

- Opens each source file
- Reads and parses the file with `tree-sitter` into an AST
- for each `Patch`:
  - Match the `Patch`'s [tree-sitter query](https://tree-sitter.github.io/tree-sitter/using-parsers#pattern-matching-with-queries) in the AST.
  - If it found something, get the equivalent C code from the `Patch`.
  - Replace the C++ code with the C equivalent.

Example:

```cpp
MI::addImm(int(10));
```

Let's say we want to patch `int(10)` to its C equivalent of `(int)(10)`.
The `Patch` for this has a `tree-sitter` query for this pattern:

```c
// The @ names elements in the query.
(call_expression // Matches a call expression.
  (primitive_type) @cast_type // Matches primitive types like int, unsigned etc.
  (argument_list) @cast_target // Matches anything within the () brackets
) @cast
```

If the `CppTranslator` finds a substring matching the pattern, it passes it as a `capture` to the `Patch`.
A `capture` is just a dictionary with the named sub-strings found. In our example it contains `cast: "int(10)"`, `cast_type: "int"` and `cast_target: "(10)"`.

Now it is trivial to concatenate sub-strings to `(int)(10)` and return it.
The `CppTranslator` now replaces `int(10)` in the source file with `(int)(10)`.

The result:

```cpp
MI::addImm((int)(10));
```

This is done with most C++ syntax. Of course, there are exceptions. Some C++ concepts are so complex to replace, we implement special scripts for them (e.g. C++ templates).
But the end-result is a source file which has very little C++ syntax left.

## Diffing

    Note: The diffing step is still unstable and is not yet reproducable.

Translating C++ files only gets so good. After all patches were applied to the file, it will likely not compile. Some syntax issues are just too difficult to fix automatically.

Fixing a handful of issues by hand again and again, is a tedious task. Especially, if you need to run the whole translation procedure multiple times.
We can hardly ask users to do the fixes by hand again every time they ran the translator.

The mechanism to solve this annoyance is diffing. It basically works as you know it from `git`.
The difference is it isn't file focused like `git` but diffs `tree-sitter` queries.
You can decide what to diff in the configuration, but let's go over it in an example:

So let's assume we update a fictional architecture. 

We have an old `<ARCH>Disassembler.c` file which might need an update.
In there we have a function which decodes an operand:

```c
void decodeOperandA(MCInst *MI, unsigned OpNum, unsigned Val) {
  if (MCInst_isPredicable(MI)) {
    MCOperand_CreateImm0(MI, Val + 1);
  }
  MCOperand_CreateImm0(MI, Val);
}
```

The same function in the _original_ C++ file looks like this:

```cpp
void decodeOperandA(MCInst &MI, unsigned OpNum, unsigned Val) {
  if (MI.isPredicable()) {
    MI.createImm(Val + 1);
  }
  MI.createImm(Val);
}
```

Now, after we ran the `CppTranslator` the result is almost valid C.
But the translation was not perfect and there is still a method invocation left:

```c
void decodeOperandA(MCInst *MI, unsigned OpNum, unsigned Val) {
  if (MI.isPredicable()) { // The isPredicable method call was not translated.
    MCOperand_CreateImm0(MI, Val + 1);
  }
  MCOperand_CreateImm0(MI, Val);
}
```

Capstone's `MCInst` struct doesn't have a callback member `isPredicable()`. So this code would not compile.
Instead, we need to replace it with the function call `MCInst_isPredicable(MI)`.

For whatever reason no `Patch` was added, and we now have to fix it by hand.
Note though that the old file (see above) already has the correct function implementation.
So instead of fixing it again by hand, we diff the previous function to the newly translated code and let the user decide what to do.

```diff
Patch: 15/230

Node: "Some Node"
+Color: NEW FILE - (Just translated)
-Color: OLD FILE - (Currently in Capstone)

⎼⎼⎼⎼⎼⎼⎼⎼⎼⎼⎼⎼⎼⎼⎼⎼⎼⎼⎼⎼⎼⎼⎼⎼⎼⎼⎼⎼⎼⎼⎼⎼⎼⎼⎼⎼⎼⎼⎼⎼⎼⎼⎼⎼⎼⎼⎼⎼⎼⎼⎼⎼⎼⎼⎼⎼⎼⎼⎼⎼⎼⎼⎼⎼⎼⎼⎼⎼⎼⎼⎼⎼⎼⎼⎼⎼⎼⎼⎼⎼⎼⎼⎼⎼⎼⎼⎼⎼⎼⎼⎼⎼⎼⎼⎼⎼⎼⎼⎼⎼⎼⎼⎼⎼⎼

void decodeOperandA(MCInst *MI, unsigned OpNum, unsigned Val) {
-	if (MCInst_isPredicable(MI)) {
+	if (MI.isPredicable()) {
  	MCOperand_CreateImm0(MI, Val + 1);
  }
	MCOperand_CreateImm0(MI, Val);
}
═════════════════════════════════════════════════════════════════════════════════════════════════════════

Choice: O, o, n, s (none) , e, p, q, ? > ?
O		- Accept ALL old diffs
o		- Accept old diff
n		- Accept new diff
e		- Edit diff (not yet implemented)
s (none) 	- Select saved choice
p		- Ignore and go to previous diff
q		- Quit (previous selections will be saved)
?		- Show this help
```

They can accept the version from the old file or accept the version from the new file.
The version from the new file would not compile, but it can be fixes by hand later.

In most cases though, the old version is the correct one, because it was fixed by someone before.

The diffing happens for each translated function which doesn't match the old code.
Of course, you can not just diff functions but any nodes in the AST of a file.
And for convenience the choices are saved as well. So if the update is run again and nothing changed, the user doesn't have to redo previous decisions.
It just automatically applies them.

This diffing step saves a lot of time.

## Adding new architectures

Adding a new architecture module works pretty much the same as above.

Generally though it gives us a standardized way of doing it. And if you know one architecture module in Capstone, you know them all.
If LLVM doesn't support your architecture you maybe find a fork which does (this is the case for TriCore or EVM).

In fact, we added two niche architectures this way. TriCore was only implemented in a fork and never upstreamed.
And the [DEC Alpha](https://en.wikipedia.org/wiki/DEC_Alpha) architecture support was dropped in LLVM 4.
We just added the `td` files again, and here we go, we have support for Alpha in Capstone.

## Last overview

To give you a last overview what components were updated and how they interact in Capstone, take a look at this diagram:

```
                                                                 ARCH_LLVM_getInstr(
                                     ARCH_getInstr(bytes)   ┌───┐   bytes)           ┌─────────┐            ┌──────────┐
                                    ┌──────────────────────►│ A ├──────────────────► │         ├───────────►│          ├────┐
                                    │                       │ R │                    │ LLVM    │            │ LLVM     │    │ Decode
                                    │                       │ C │                    │         │            │          │    │ Instr.
                                    │                       │ H │                    │         │decode(Op0) │          │◄───┘
┌────────┐ disasm(bytes) ┌──────────┴──┐                    │   │                    │ Disass- │ ◄──────────┤ Decoder  │
│CS Core ├──────────────►│ ARCH Module │                    │   │                    │ embler  ├──────────► │ State    │
└────────┘               └─────────────┘                    │ M │                    │         │            │ Machine  │
                                    ▲                       │ A │                    │         │decode(Op1) │          │
                                    │                       │ P │                    │         │ ◄──────────┤          │
                                    │                       │ P │                    │         ├──────────► │          │
                                    │                       │ I │                    │         │            │          │
                                    │                       │ N │                    │         │            │          │
                                    └───────────────────────┤ G │◄───────────────────┤         │◄───────────┤          │
                                                            └───┘                    └─────────┘            └──────────┘
```

The `Capstone Core`, `Arch Module` and `Arch_Mapping` provide the API to the LLVM disassembler logic.
We have not spoken about those because they are irrelevant for the topic of generating disassemblers.

The two boxes on the right, are the code copies from LLVM, which do the actual decoding work.
The `LLVM Disassembler` component decodes single operands and handles special cases. This one was translated by the `CppTranslator`.
While the `LLVM Decoder State Machine` was generated by our patched LLVM backends.

The same structure applies to the printing of the asm text. Though we have only scratched this here for the sake of brevity.

## Wrap up

If one looks at the whole update procedure, it is still a rather complicated. But the result is worth it.

The amount of time someone has to spend for updating an architecture module in Capstone went down from "no one did it" to roughly 6-29 hours.
To update the ARM architecture module to LLVM 16 for example, the times were:

1. Rebasing patched backends to new LLVM release = ~1-3h
2. Running the update scripts and diffing = 5min - 1h
3. Fixing rest of build errors by hand = ~30min - 5h
4. Handle new operands on the CS side (filling the detail info and tests) - 3-10h
5. Bug fixing - 2h-10h

(Please be aware though, that the time estimates from above don't include the "read into" time someone has to spend).

The lower estimates are for small changes on the LLVM side, the upper for many changes (spanning multiple LLVM releases).

Here is process described above in action:

[![asciicast](https://asciinema.org/a/kwoEWhkWMQ4xBvh9o6WZ9632r.svg)](https://asciinema.org/a/kwoEWhkWMQ4xBvh9o6WZ9632r)

## Future plans

We have a list of features and architectures which will come.
Most architectures are already updated, but `ARC`, `BPF` and `SPARC` are still on [the list](https://github.com/capstone-engine/capstone/issues/2015).

While working on the updater we found many shortcomings or flaws in the target definitions in LLVM.
Those will be upstreamed to LLVM eventually.

In the very long run we would like to participate with the LLVM folks in redesign the TableGen backends.
It would be nice to have the problems solved, which we mentioned above.

And of course, if you want to update now an already present Capstone module or add support for a new one, feel free to drop a message in [issue #2015](https://github.com/capstone-engine/capstone/issues/2015).
We are happy to hear about it and will guide you through the process.

## References

- [Auto-Sync progress issue](https://github.com/capstone-engine/capstone/issues/2015)
- [Auto-Sync documentation](https://github.com/capstone-engine/capstone/blob/next/docs/AutoSync.md)
- [TableGen documentation](https://llvm.org/docs/TableGen/)
- [Capstone's LLVM fork](https://github.com/capstone-engine/llvm-capstone)
