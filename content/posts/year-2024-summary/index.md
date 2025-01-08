---
author: "xvilka"
title: "2024 Year Summary"
date: "2025-01-02"
summary: "An overview of the work done in 2024"
tags: ["rizin", "cutter", "capstone", "rzil"]
ShowToc: false
weight: 2
---

This year we focused mainly on the "backbone" of the Rizin framework and all related tools, including Cutter. This will become a foundation of the future work we plan to finish in 2025. The major goal is to release 0.8.0 in upcoming months. As for the longer term you can see [our roadmap](https://rizin.re/roadmap/) for details.

## Releases

- Rizin versions 0.7.x and Cutter 2.3.3-2.3.4
- 80% of work is done for [Rizin 0.8.0](https://github.com/rizinorg/rizin/milestone/18)

## Capstone

A bulk of our effort was spent towards Capstone improvements as it is the core dependency and the main disassembly engine for many architectures supported by Rizin. A long-going project called Auto-Sync was finally merged and largely completed. You could read more details in our [corresponding article](https://rizin.re/posts/auto-sync/).

- Updated PPC, ARM, AArch64 and SystemZ to LLVM 18.
- Tricore support by billow
- HPPA by r3v0lt
- Alpha by r3v0lt
- ARC PR by r3v0lt (not yet merged: [#2570](https://github.com/capstone-engine/capstone/pull/2570))
- MIPS, microMIPS, and nanoMIPS support by deroad
- Xtensa by billow

## Rizin

- Started merging RzAsm and RzAnalysis for the future RzArch
- MIPS update to a Capstone-based plugin by deroad
- Basic LoongArch support by deroad
- PIC MCU family RzIL uplifting by billow
- MSP430 RzIL uplifting by moste00
- Xtensa RzIL uplifting by billow
- Hexagon RzIL uplifting by Rot127
- Finished (almost, with one PR still not yet merged) conversion to the rzshell
- Rewritten text representation of ELF, PE, NE relocations by Roeegg2
- Added initial support for the Alpha architecture
- Many bugfixes, refactorings, and performance optimizations

## Cutter

- Switch to the Qt6 by default, including for forming releases
- Updated Rizin

## Other projects

- rz-ghidra: add support for PIC architectures
- rz-ghidra: add support for Tricore architecture

## Miscellaneous

- [Google Summer of Code 2024](https://summerofcode.withgoogle.com/programs/2024/organizations/rizin)
- [FOSSAsia 2024 conference](https://eventyay.com/e/55d2a466/session/9017)
