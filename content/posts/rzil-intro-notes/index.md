---
author: "brightprogrammer"
title: "RzIL Introductory Notes"
date: 2023-04-10T20:49:41+05:30
summary: "An Introduction To Rizin with Examples"
tags: ["rizin", "rzil"]
ShowToc: false
weight: 2
---

### References First

- Some idea about how RzIL works can be found here : [https://github.com/rizinorg/rizin/blob/dev/doc/rzil.md](https://github.com/rizinorg/rizin/blob/dev/doc/rzil.md)
- RzIL is based on [**BAP's Core Theory**](http://binaryanalysisplatform.github.io/bap/api/odoc/bap-core-theory/Bap_core_theory/index.html) so do read about that too.
- Analysis code is present in [`librz/include/rz_analysis.h`](https://github.com/rizinorg/rizin/tree/dev/librz/include/rz_analysis.h) and [`librz/analysis`](https://github.com/rizinorg/rizin/tree/dev/librz/analysis).  
- ESIL uplifters are present in [`librz/analysis/p`](https://github.com/rizinorg/rizin/tree/dev/librz/analysis/p) and the new RzIL uplifters are present in [`librz/analysis/arch`](https://github.com/rizinorg/rizin/tree/dev/librz/analysis/arch).  
- Each of these intermediate languages can be emulated so there must be a VM to interpret these ILs and run them. That code can be found in [`librz/analysis/il`](https://github.com/rizinorg/rizin/tree/dev/librz/analysis/il).  
- RzIL related headers can be found in [`librz/include/rz_il`](https://github.com/rizinorg/rizin/blob/dev/librz/include/rz_il) and definitions can be found in [`librz/il`](https://github.com/rizinorg/rizin/tree/dev/librz/il)  
- RzIL Opcodes are declared in [`rz_il_opcodes.h`](https://github.com/rizinorg/rizin/blob/dev/librz/include/rz_il/rz_il_opcodes.h) and defined in [`il_opcodes.c`](https://github.com/rizinorg/rizin/tree/dev/librz/il/il_opcodes.c)  
- There are multiple helper macros to help writing uplifters easily defined in [`rz_il_opbuilder_begin.h`](https://github.com/rizinorg/rizin/blob/dev/librz/include/rz_il/rz_il_opbuilder_begin.h)  

### Pure & Effect Operations in RzIL  

- `RzILOpEffect` is to reperesent an instruction/operation that changes the VM state from one to another. This means instructions like **`ADD`**, **`SUB`** etc… should be treated as `RzILOpEffect`! But that’s not the complete story, keep reading…  
- `RzILOpPure` is to represent operations/instructions that don’t change the CPU state but evaluate to a pure expression. By pure, we mean, it will be a constant expression. So the result of instructions like **`ADD`**, **`SUB`**, etc… can now be treated like an `RzILOpPure`! There are three different types of `RzILOpPure` operations, meaning three different types of operations in RzIL that evaluate to three completely different types of values :  
    - `RzILOpBool` To represent a boolean pure expression.  
    - `RzILOpBitvector` To represent integers/numerical values of any size!  
    - `RzILOpFloat` To represent float values of different spec/fomat.  
    You can find all these definitions in [`rz_il_opcodes.h#L41`](https://github.com/rizinorg/rizin/blob/9ab709bc34843f04ffde3b63322c809596123e77/librz/include/rz_il/rz_il_opcodes.h#L41)  

This means that an instruction like **`ADD`** is not just the instruction **`ADD`**! So, to make this more clear, I’ll take examples :  
```c
// This is an x64 instruction lifter implemented for the ADD instruction
IL_LIFTER(add) {
        RzILOpEffect * op1 = SETL("op1", x86_il_get_op(0));
        RzILOpEffect * op2 = SETL("op2", x86_il_get_op(1));
        RzILOpEffect * sum = SETL("sum", ADD(VARL("op1"), VARL("op2")));

        RzILOpEffect * set_dest = x86_il_set_op(0, VARL("sum"));
        RzILOpEffect * set_res_flags = x86_il_set_result_flags(VARL("sum"));
        RzILOpEffect * set_arith_flags = x86_il_set_arithmetic_flags(VARL("sum"), VARL("op1"), VARL("op2"), true);

        return SEQ6(op1, op2, sum, set_dest, set_res_flags, set_arith_flags);
}
```

To understand this, you first need to understand the **semantics** of the instruction that you want to lift. In our case that’s the **`ADD`** instruction.  

The **`ADD`** instruction takes two arguments in x86, **`ADD reg/mem, reg/mem/imm`**. But that’s just the sytax of the **`ADD`** instruction. If we try to understand what this operation/instuction is doing (hence the semantics) we’ll notice these points:  
- **`ADD`** is a binary operation so it takes two arguments to perform addition. You can notice that in the first two lines of code in `IL_LIFTER(add)` function.  
- **`ADD`** instruction must also store it’s result somewhere! In this case it’s the first operand but other architectures like MIPS, RISCV etc.. specify an explicit operand to use to store the result. You can see this third operand as the `sum` as destination operand.  
- `op1`, `op2` and `op3` are actually `RzILOpPure` objects here. How? Let’s take a look at how this whole system works.  
    ```c
    // definition in `librz/include/rz_il_opbuilder_begin.h`
    #define VARL(name) rz_il_op_new_var(name, RZ_IL_VAR_KIND_LOCAL)
    
    // definition in `librz/il/il_opcodes.c`
    RZ_API RZ_OWN RzILOpPure * rz_il_op_new_var(RZ_NONNULL
            const char * v, RzILVarKind kind) {
            rz_return_val_if_fail(v, NULL);
            RzILOpPure * ret;
            rz_il_op_new_2(Pure, RZ_IL_OP_VAR, RzILOpArgsVar,
                    var, v, kind);
            return ret;
    }
    ```
    
This will create a new `RzILOpPure` with it’s name as `name` and set it’s kind as `LOCAL` or `GLOBAL`. In case of add operation, registers are used locally unlike a globally defined memory region, so the type is `RZ_IL_VAR_KIND_LOCAL`.  
    
- Loading of operands from register is an `RzILOpEffect` operation! Why? Well don’t think of this like execution in a normal CPU, we are being emulated in a VM so any such instruction (read/store from register/memory) will be treated as a VM state changing instruction and hence it’s an effect!  
- Hence first three lines are creating an `RzILOpEffect` and not `RzILOpPure` operation. All `OpPure`s are hidden by the use of `VARL`.  
- Now the **`ADD`** instruction not only adds given operands and stores in a destination operand, but also updates a status/flags register. This is also an effect since it’s changing the VM CPU state! So, we again have three more effects.  
- Now all of this can be treated as a sequence operations to be performed to properly emulate/execute a single **`ADD`** instruction on whatever architecture you uplifted for. Hence we have the last line of `IL_LIFTER(add)` forming a sequence of these effect operations.  

So, this is how you capture the semantics of a single instruction and uplift it!  

Now, let’s take example of another instruction from same architecture:  
```c
/**
 * DIV
 * Unsigned division
 * One operand (memory address), used as the divisor
 */
IL_LIFTER(div) {
        RzILOpEffect * ret = NULL;

        switch (ins -> structure -> operands[0].size) {
        case 1: {
                /* Word/Byte operation */
                RzILOpEffect * ax = SETL("_ax", x86_il_get_reg(X86_REG_AX));
                RzILOpEffect * temp = SETL("_temp", UNSIGNED(8, DIV(VARL("_ax"), VARL("_src"))));

                RzILOpPure * cond = UGT(VARL("_temp"), UN(8, 0xff));
                RzILOpEffect * else_cond = SEQ2(x86_il_set_reg(X86_REG_AL, VARL("_temp")), x86_il_set_reg(X86_REG_AH, MOD(VARL("_ax"), VARL("_src"))));

                ret = SEQ3(ax, temp, BRANCH(cond, NULL, else_cond));
                break;
        }
        case 2: {
                /* Doubleword/Word operation */
                RzILOpEffect * dxax = SETL("_dxax", LOGOR(SHIFTL0(UNSIGNED(32, x86_il_get_reg(X86_REG_DX)), UN(8, 16)), UNSIGNED(32, x86_il_get_reg(X86_REG_AX))));
                RzILOpEffect * temp = SETL("_temp", UNSIGNED(16, DIV(VARL("_dxax"), VARL("_src"))));

                RzILOpPure * cond = UGT(VARL("_temp"), UN(16, 0xffff));
                RzILOpEffect * else_cond = SEQ2(x86_il_set_reg(X86_REG_AX, VARL("_temp")), x86_il_set_reg(X86_REG_DX, MOD(VARL("_dxax"), VARL("_src"))));

                ret = SEQ3(dxax, temp, BRANCH(cond, NULL, else_cond));
                break;
        }
        case 4: {
                /* Quadword/Doubleword operation */
                RzILOpEffect * edxeax = SETL("_edxeax", LOGOR(SHIFTL0(UNSIGNED(64, x86_il_get_reg(X86_REG_EDX)), UN(8, 32)), UNSIGNED(64, x86_il_get_reg(X86_REG_EAX))));
                RzILOpEffect * temp = SETL("_temp", UNSIGNED(32, DIV(VARL("_edxeax"), VARL("_src"))));

                RzILOpPure * cond = UGT(VARL("_temp"), UN(32, 0xffffffff ULL));
                RzILOpEffect * else_cond = SEQ2(x86_il_set_reg(X86_REG_AX, VARL("_temp")), x86_il_set_reg(X86_REG_DX, MOD(VARL("_edxeax"), VARL("_src"))));

                ret = SEQ3(edxeax, temp, BRANCH(cond, NULL, else_cond));
                break;
        }
        case 8: {
                /* Doublequadword/Quadword operation */
                RzILOpEffect * rdxrax = SETL("_rdxrax", LOGOR(SHIFTL0(UNSIGNED(128, x86_il_get_reg(X86_REG_RDX)), UN(8, 64)), UNSIGNED(128, x86_il_get_reg(X86_REG_RAX))));
                RzILOpEffect * temp = SETL("_temp", UNSIGNED(64, DIV(VARL("_rdxrax"), VARL("_src"))));

                RzILOpPure * cond = UGT(VARL("_temp"), UN(64, 0xffffffffffffffff ULL));
                RzILOpEffect * else_cond = SEQ2(x86_il_set_reg(X86_REG_AX, VARL("_temp")), x86_il_set_reg(X86_REG_DX, MOD(VARL("_rdxrax"), VARL("_src"))));

                ret = SEQ3(rdxrax, temp, BRANCH(cond, NULL, else_cond));
                break;
        }
        default:
                RZ_LOG_ERROR("RzIL: x86: DIV: Invalid operand size\n");
                return NULL;
        }

        /* We need the divisor to be as wide as the operand, since the sizes of the dividend and the divisor need to match */
        RzILOpEffect * op = SETL("_src", UNSIGNED(ins -> structure -> operands[0].size * 2 * BITS_PER_BYTE, x86_il_get_op(0)));

        /* Check if the operand is zero, return NULL if it is (to avoid divide by zero) */
        return SEQ2(op, BRANCH(IS_ZERO(VARL("_src")), NULL, ret));
}
```

Please allow me to jog your memory with syntax of **`DIV`** instruction first : 
![div-instr](/images/div-instr-table.png)

Now, the above code does following things : 

- In different cases, very similar things are done since only the size of operation is changing.
- Now before you start reading from top, take a look at the `_src` `RzILOpPure` created in last line and loading value into `_src` in second last line as an effect.
- Now, in each of the cases, instruction like this is present :
    
    ```c
    RzILOpEffect *dxax = SETL("_dxax", LOGOR(SHIFTL0(UNSIGNED(32, x86_il_get_reg(X86_REG_DX)), UN(8, 16)), UNSIGNED(32, x86_il_get_reg(X86_REG_AX))));
    ```
    
    This is getting the value from registers and shifting and taking or. Please take a look at how defines like `LOGOR`, `SHIFTR0`, `SHIFTL0` etc are defined.
    
    Since this is again a load operation, this will be treated as an effect.
    
- But, notice there’s one direct `RzILOpPure` this time in the code! All division operations must be checked whether the divide resulted in some invalid result and for that we have
    
    ```c
    RzILOpPure *cond = UGT(VARL("_temp"), UN(16, 0xffff));
    ```
    
    This is condition for **Unsigned Greater Than (UGT)**. Notice also that here is the definition of pure value `_temp`.
    

Now, you can understand basic instuction lifters, but we won’t know until unless we actually write one!

### Understanding Already Present Uplifters

All already present uplifters have atleast two functions in their head files as an `RZ_API` (meaning to be exposed to and used by other modules)

```c
// 8051 uplifter
RZ_IPI I8051Op *rz_8051_op_parse(RZ_NONNULL RzAnalysis *analysis, RZ_NONNULL const ut8 *buf, int len, ut64 pc);
RZ_IPI RzILOpEffect *rz_8051_il_op(RZ_NONNULL RzAnalysis *analysis, RZ_NONNULL const ut8 *buf, int len, ut64 pc);
RZ_IPI RzAnalysisILConfig *rz_8051_il_config(RZ_NONNULL RzAnalysis *analysis);

// x86 uplifter
RZ_IPI bool rz_x86_il_opcode(RZ_NONNULL RzAnalysis *analysis, RZ_NONNULL RzAnalysisOp *aop, ut64 pc, RZ_BORROW RZ_NONNULL const X86ILIns *ins);
RZ_IPI RzAnalysisILConfig *rz_x86_il_config(RZ_NONNULL RzAnalysis *analysis);

// arm uplifter
RZ_IPI RzILOpEffect *rz_arm_cs_32_il(csh *handle, cs_insn *insn, bool thumb);
RZ_IPI RzAnalysisILConfig *rz_arm_cs_32_il_config(bool big_endian);
RZ_IPI RzILOpEffect *rz_arm_cs_64_il(csh *handle, cs_insn *insn);
RZ_IPI RzAnalysisILConfig *rz_arm_cs_64_il_config(bool big_endian);
```

Notice how each of this have a function to return `RzILOpEffect` and `RzAnalysisILConfig`?

By this point, we already have some good information about how `RzILOpEffect` works! `RzAnalysisILConfig` is to specify initial state of RzIL VM. This contains details like names of registers, architecture bit size etc…  
Here’s the definition of [`RzAnalysisILConfig`](https://github.com/rizinorg/rizin/blob/9ab709bc34843f04ffde3b63322c809596123e77/librz/include/rz_analysis.h#L1107)  
```c
/**
 * \brief Description of the global context of an RzAnalysisILVM
 *
 * This defines all information needed to initialize an IL vm in order to run
 * in a declarative way, in particular:
 *
 * * Size of the program counter: given explicitly in `pc_size`
 * * Endian: given explicitly in `big_endian`
 * * Memories: currently always one memory with index 0 bound against IO, with key size given by `mem_key_size` and value size of 8
 * * Registers: given explicitly in `reg_bindings` or derived from the register profile with `rz_il_reg_binding_derive()`
 * * Labels: given explicitly in `labels`
 * * Initial State of Variables: optionally given in `init_state`
 */
typedef struct rz_analysis_il_config_t {
        ut32 pc_size; ///< size of the program counter in bits
        bool big_endian;
        /**
         * Optional null-terminated array of registers to bind to global vars of the same name.
         * If not specified, rz_il_reg_binding_derive will be used.
         */
        RZ_NULLABLE
        const char ** reg_bindings;
        ut32 mem_key_size; ///< address size for memory 0, bound against IO
        RzPVector /*<RzILEffectLabel *>*/ labels; ///< global labels, primarily for syscall/hook callbacks
        RZ_NULLABLE RzAnalysisILInitState * init_state; ///< optional, initial contents for variables/registers, etc.
        // more information might go in here, for example additional memories, etc.
} RzAnalysisILConfig; 
```

To look into this in more detail, search for it in [`librz/include/rz_analysis.h`](https://github.com/rizinorg/rizin/blob/dev/librz/include/rz_analysis.h)

So we need only two main things to write an uplifter (once we are able to get the disassembly) : 

- A function to convert given instruction data to it’s corresponding IL in RzIL.
- A function to specify initial state of VM, like registers, arch bits, what the registers and memory contain initially.

