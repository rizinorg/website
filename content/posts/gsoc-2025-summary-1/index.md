---
author: "xvilka"
title: "Google Summer of Code 2025 Summary - Part 1"
date: "2025-09-21"
summary: ""
tags: ["rizin", "gsoc"]
ShowToc: true
TocOpen: false
weight: 2
---

# Google Summer of Code 2025 Summary

Two-thirds of GSoC 2024 have officially concluded, and we are pleased to congratulate two participants who successfully passed the program and completed the most important parts of their projects. One participant is [continuing work on object-oriented (C++, Objective-C, Swift, etc.) analysis](https://rizin.re/posts/gsoc-2025-announcement/#tushar-jain) due to the extended timeline of the task.

## Taksh: Refactoring Rizin's Native Debug Plugin – My GSoC 2024 Journey

Hello, I'm Taksh (aka well-mannered-goat), a junior student at IIIT Gwalior. I'm passionate about systems programming and enjoy building projects such as DNS servers and virtual machines. You can find me on [GitHub](https://github.com/well-mannered-goat) and [LinkedIn](https://www.linkedin.com/in/taksh-patel-a78b531a8/).

While searching for software written in C, I discovered Rizin and became fascinated with it. Through Rizin, I developed my passion for systems programming. This summer, I worked with Rizin to refactor and fix their native debug module.

### The Challenge: Understanding Rizin's Debugger

Rizin's debugger can be utilized through various plugins, including the native plugin, GDB plugin, and WinDbg plugin. The native debugger plugin is unique in its support for multiple architectures and operating systems. This broad compatibility makes the codebase challenging to understand and maintain. Due to its monolithic design, fixing the native plugin for one platform often caused issues on others. Testing became particularly difficult, as validating changes across different platforms after each refactor was complex, leaving the debug module largely untested.

Initially, we proposed rewriting the entire native debug plugin. However, since this would be an enormous undertaking with potential immediate issues, we pivoted to refactoring and fixing the existing implementation.

### Test and Fix

My work began with adding testing support for architectures and operating systems supported by Rizin's CI:

- [Linux - PPC64LE](https://github.com/rizinorg/rizin/pull/5184)
- [Linux - ARM64](https://github.com/rizinorg/rizin/pull/5232)
- [NetBSD - x86_64](https://github.com/rizinorg/rizin/pull/5208)
- [FreeBSD - x86_64](https://github.com/rizinorg/rizin/pull/3273)
- [OpenBSD - x86_64](https://github.com/rizinorg/rizin/pull/5226)

Through this work, I gained deep insights into how debuggers function and learned about the different system calls they utilize. I also discovered the distinctions between BSD and Linux systems and their operational differences.

This work included correcting register profiles according to how ptrace uses them. For example, I had to adjust the NetBSD profile to match how ptrace sends register value data using the [struct reg](https://github.com/NetBSD/src/blob/a037e5e73c651f506fa7a32005df360c5f840f99/sys/arch/amd64/include/reg.h#L51).

I also utilized [sysctl](https://man.netbsd.org/sysctl.3) to retrieve process information such as maps, state, and signal data in BSD systems.

### Refactoring the Debug Native Plugin

The native plugin was compiled using predefined compiler macros for CPU architectures and operating systems. This approach hindered the developer experience and made refactoring the debug module challenging.

Here's a simple example demonstrating the code transformation:

#### Before:

```c
#if __arm64__
#if __LINUX__
// native debug plugin code for Linux-arm64 platform
#elif __BSD__
// native debug plugin code for BSD-arm64 platforms
#endif
#elif __i386__
#if __LINUX__
// native debug plugin code for Linux-x86 platform
#elif __BSD__
// native debug plugin code for BSD-x86 platforms
#endif
#endif
```

#### After:
![Compilation-native_debug_plugin](https://hackmd.io/_uploads/rkJaCH_oll.png)

*Compilation of native debugger plugin in Rizin*

This refactoring significantly improves platform-specific bug fixing and feature development for the native debugger. Changes to one platform no longer cause bugs in others, which was a persistent issue previously.

### Signal Handling Improvements

In the Rizin debugger, we track process behavior and report why a process stops using `RZ_DEBUG_REASON_*` constants. When a process hits a breakpoint, it receives `SIGTRAP`. However, this was incorrectly caught as `RZ_DEBUG_REASON_SIGNAL` rather than `RZ_DEBUG_REASON_BREAKPOINT`, causing problems in the debugger, particularly on BSD systems. This required significant refactoring.

Key documentation that aided my work included [ptrace](https://man7.org/linux/man-pages/man2/ptrace.2.html), [waitpid](https://linux.die.net/man/2/waitpid), [kill](https://man7.org/linux/man-pages/man2/kill.2.html), and [sysctl](https://man.netbsd.org/sysctl.3), depending on the operating system.

I worked with these systems using QEMU and running Rizin on virtual machines.

Some work still in progress includes:
- [Refactoring the entire native plugin](https://github.com/rizinorg/rizin/pull/5295) – improving code readability and maintainability
- [Adding support for Linux s390x system](https://github.com/rizinorg/rizin/pull/5267)

### Challenges Faced and Solutions

#### 1. OpenBSD CI Run
Due to OpenBSD's secure nature, process maps cannot be easily fetched unless `kern.securelevel` is set to 0. These maps are essential for obtaining information about the child process (debuggee). Therefore, Rizin needed to run with permissions using doas in the OpenBSD CI.

#### 2. Refactoring the Entire Native Plugin
The native plugin relied heavily on preprocessor macros like `#if` and `#elif` to separate platform-specific code using predefined architecture and operating system compiler macros. This made understanding the code difficult when working on a single platform. I separated the code by platform and modified the build system to compile platform-specifically rather than relying on macros.

#### 3. Adding Linux s390x Support
The base address for the debuggee is set 4 bytes behind due to missing ELF magic bytes, while the marshal code assumes the ELF magic bytes are already present. I'm currently investigating this issue, which appears to be related to ptrace behavior in s390x systems and differences in load segments between Linux s390x and other Linux systems.

#### 4. Signal Handling
Signals can originate from either the kernel or user space. Kernel-generated signals are triggered by specific instructions. To skip signal handlers, we skip the triggering instruction after the first signal occurrence to prevent repetition. However, if the signal is user-generated, we incorrectly skip an instruction unnecessarily. The solution involves marking signals as kernel or user-generated after receiving signal information via ptrace.

### What's Next?

The refactoring and testing process revealed numerous previously unnoticed bugs, including:
1. [Signal Handler Skipping Logic](https://github.com/rizinorg/rizin/issues/5332)
2. [Fixing Coredump Generation of Linux-ARM64 Debugger](https://github.com/rizinorg/rizin/issues/5331)

The refactoring work continues. While we've established a solid foundation with proper testing infrastructure, complete modularization of the native plugin remains an ongoing effort.

### Conclusion

This summer was an incredible journey. Going from being completely new to systems programming to refactoring a multi-architecture, multi-OS debugger provided invaluable learning experiences. Some days were challenging, and others were chaotic (including crashing my own system from running multiple VMs for extended periods).

Special thanks to [xvilka](https://github.com/notxvilka), [deroad](https://github.com/wargio), and [Rot127](https://github.com/Rot127) for their guidance and support. I also thank the GSoC team for this opportunity.

The journey continues – there's always more to learn!

---

## Ahmed Kamal: Thread Safety Improvement

Hi, I'm Ahmed Kamal, a GSoC 2024 contributor at Rizin. I'm a Computer Engineering student at Cairo University and a part-time DevOps engineer. You can find me on [GitHub](https://www.github.com/ahmed-kamal2004) and [LinkedIn](https://www.linkedin.com/in/ahmed-kamal-649b4a231).

I'm passionate about open source and enjoy contributing to projects while leaving my mark in the open source community. Through GSoC and Rizin, I've worked on a crucial project: **Thread Safety Improvement**.

### Why This Project Is Essential for Rizin

Rizin's codebase has historically relied heavily on static and global variables. This design choice has become a major obstacle to implementing multi-threading, limiting performance improvements and preventing seamless integration with Cutter.

The reliance on shared state prevents safe parallel thread execution, introduces race conditions, and necessitates heavy locking mechanisms that undermine scalability. Consequently, Cutter cannot fully leverage Rizin as a responsive backend, negatively impacting the user experience.

Eliminating global/static state and encapsulating functionality within well-defined contexts is a critical step forward. This approach enables true multi-threading, improves responsiveness, and makes Rizin more extensible, ultimately delivering a better, faster, and more seamless experience for Cutter users.

### Problem Explanation


All threads within a process share the same data segment where global and static variables are stored. This means every thread accesses and modifies the same global/static variables, leading to race conditions and unsafe concurrent behavior.

In contrast, contexts work safely because they're allocated on the stack (per thread) or allocate separate heap memory for each thread with their own copy. Each thread holds its own context instance, keeping data private unless explicitly shared. This eliminates accidental interference, reduces heavy locking requirements, and creates a safer system without race conditions.

### Project Goals

The following list outlines the major project goals:

- **RzUtil** (`path.c`): Replace static variables with RzSys and RzPath context structures; remove constructor/destructor functions ([PR #5148](https://github.com/rizinorg/rizin/pull/5148))
- **RzCore** (`agraph.c`): Encapsulate static state into a non-exposed AGraphContext ([PR #5207](https://github.com/rizinorg/rizin/pull/5207))
- **Update libmagic** based on openbsd/src@0e59d0d: Update library and eliminate static variables ([PR #5288](https://github.com/rizinorg/rizin/pull/5288))
- **RzIO Plugins** (`io_wingbg`, `io_self`, `io_qnx`, `io_plugin`, `io_gdb`): Create dedicated contexts per plugin for managing plugin-specific state ([PR #5237](https://github.com/rizinorg/rizin/pull/5237), [PR #5241](https://github.com/rizinorg/rizin/pull/5241), [PR #5252](https://github.com/rizinorg/rizin/pull/5252), [PR #5260](https://github.com/rizinorg/rizin/pull/5260), [PR #5249](https://github.com/rizinorg/rizin/pull/5249))
- **RzAsm Plugins** (`cris`, `vax`, `lanai`, `arc`, `hppa`, `z80`): Create dedicated contexts per plugin ([PR #5302](https://github.com/rizinorg/rizin/pull/5302), [PR #5325](https://github.com/rizinorg/rizin/pull/5325))
- **RzBin Plugins** (`sms`, `avr`, `z64`): Create dedicated contexts per plugin ([PR #5222](https://github.com/rizinorg/rizin/pull/5222), [PR #5223](https://github.com/rizinorg/rizin/pull/5223), [PR #5210](https://github.com/rizinorg/rizin/pull/5210))
- **WASM analysis**: Replace static/global variables with contexts ([PR #5192](https://github.com/rizinorg/rizin/pull/5192))
- **RzLang**: Replace static/global variables with contexts
- **RzEgg**: Replace static/global variables with contexts ([PR #5212](https://github.com/rizinorg/rizin/pull/5212))
- **RzCons** (`cons.c`, `output.c`): Carefully refactor static state while preserving Windows-specific behaviors through testing ([PR #5314](https://github.com/rizinorg/rizin/pull/5314))
- **RzCore Terminal UI** (`./librz/core/tui/`): Encapsulate static state for multi-threaded TUI operation ([PR #5269](https://github.com/rizinorg/rizin/pull/5269))
- **SDB**: Remove unused static-variable-dependent functions; refactor necessary ones to use context structures ([PR #5276](https://github.com/rizinorg/rizin/pull/5276), [PR #5278](https://github.com/rizinorg/rizin/pull/5278))
- **Additional improvements** ([PR #5219](https://github.com/rizinorg/rizin/pull/5219), [PR #5286](https://github.com/rizinorg/rizin/pull/5286), [PR #5307](https://github.com/rizinorg/rizin/pull/5307), [PR #5326](https://github.com/rizinorg/rizin/pull/5326))

### My Work

This project required deep understanding of the Rizin codebase. To make precise and effective changes, I thoroughly studied its architecture, including the core, utils, sdb, magic, and plugin subsystems (analysis, IO, ASM, bin, etc.).

Over three months, I submitted numerous PRs focused on removing static and global variables and replacing them with contexts and structs. This required significant refactoring to ensure proper functionality with the new design. We validated these changes through extensive testing, including unit, integration, and regression tests across multiple CI workflows on GitHub Actions, Travis CI, TinyCC, and local environments. This process confirmed that the refactor worked as expected while improving thread safety throughout the codebase.

### Challenges

Working on this project presented several challenges that fostered both technical and personal growth:

#### Understanding a Large, Complex Codebase
Rizin is a mature and extensive project. Gaining deep understanding of how components like core, util, sdb, cmd, and plugins interact required significant time and effort.

#### Dealing with Static and Global Dependencies
Static and global variables were deeply embedded across multiple layers. Refactoring them without breaking functionality required extreme caution, careful analysis, and incremental changes to preserve functionality.

#### Maintaining Cross-Platform Compatibility
Since Rizin supports Linux, macOS, FreeBSD, OpenBSD, QNX, and Windows, every refactor required validation across multiple environments. Windows presented unique challenges due to platform-specific behaviors and API differences.

#### Updating libmagic
One of the most challenging tasks was updating libmagic. Rizin relied on an outdated file implementation from OpenBSD (2009), lacking modern improvements. Updating to the 2015 implementation required carefully merging years of upstream changes while ensuring compatibility with Rizin's existing codebase. This wasn't a straightforward replacement – I had to resolve API differences, adapt Rizin's integration to the new structure, and verify consistent behavior across all supported platforms. The process demanded extensive testing and debugging but ultimately removed static dependencies, improved maintainability, and aligned Rizin with a more modern and secure upstream library.

### Outcome

As a result of this project, the vast majority of Rizin's codebase is now thread-safe. Only a few components remain, and the mentors will decide whether to refactor, remove, or keep them unchanged. This work brought Rizin to an important milestone: the codebase is now over 90% thread-safe.

### Future Work

Moving forward, the remaining parts of the codebase should be discussed with the Rizin community to determine whether they should be removed, redesigned, or refactored. This collaboration will help push Rizin closer to becoming 100% thread-safe.

I deeply appreciate the Rizin community – it's unique, supportive, and filled with talented people. I'm proud to be part of it and plan to continue contributing to the Rizin codebase and infrastructure beyond GSoC. Many exciting changes in the Rizin ecosystem are expected in the near future, and I'll do my best to be part of them.

Thanks to [@deroad](https://github.com/wargio), [@xvilka](https://github.com/notxvilka), [@Rot127](https://github.com/Rot127), and the entire community.
