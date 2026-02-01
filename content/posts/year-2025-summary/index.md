---
author: "xvilka"
title: "2025 Year Summary"
date: "2026-02-01"
summary: "An overview of the work done in 2025"
tags: ["rizin", "cutter", "capstone", "rzil"]
ShowToc: false
weight: 2
---

# 2025 summary

This year we focused on finishing the RzIL migration (the work still continues in 2026) and the backbone of the Rizin subsystem

## Team

A new member joined the Rizin Core community - [billow](https://github.com/b1llow).

## Releases

- Rizin versions 0.8.x and Cutter 2.4.x
- 60% of work is done for Rizin 0.9.0

## Capstone

Just like the last year, we worked on some important improvements in Capstone that are necessary for Rizin:

- Updated SPARC support for the latest by rot127
- Updated and greatly extended RISC-V support (with extensions) by moste00
- Various fixes in existing architectures

## Rizin

- SPARC RzIL uplifting was implemented by Rot127
- H8/300 RzIL uplifting was implemented by billow
- MIPS RzIL uplifting was implemented by deroad
- RzSearch and search commands were rewritten from scratch and covered by tests
- Various UI/UX improvements in Rizin commands - colorization, clearer output
- MIPS support was greatly improved by rewriting analysis plugin and ELF, PE, COFF handling
- C++ demangler was rewritten from scratch to support both v2 and v3 mangling schemes in one code: [PR #77](https://github.com/rizinorg/rz-libdemangle/pull/77), not yet merged
- x86 plugin was migrated from Capstone to Zydis
- Lua support was rewritten and upgraded to support all Lua versions from 5.0 to 5.5: [PR #5555](https://github.com/rizinorg/rizin/pull/5555), not yet merged
- Python bytecode support was extended to handle also 3.11-3.14 versions
- H8/500 support by billow
- Infineon C166 support: [PR #5309](https://github.com/rizinorg/rizin/pull/5309), not yet merged
- RISC-V plugin switched to Capstone: [PR #5482](https://github.com/rizinorg/rizin/pull/5482), not yet merged
- Search of cryptographic was rewritten to support more algorithms
- Native debugger was refactored and covered by tests for all architectures and platforms we have on our CI [(Link to GSoC report)](https://rizin.re/posts/gsoc-2025-summary-1/#taksh-refactoring-rizins-native-debug-plugin--my-gsoc-2025-journey)
- Better handling of relocations, implement missing relocations (not yet complete, work still continues in 2026)
- More globals and statics were removed [(Link to GSoC report)](https://rizin.re/posts/gsoc-2025-summary-1/#ahmed-kamal-thread-safety-improvement)
- Improved C++ class analysis (not yet complete, more work is needed)
- ROP search was greatly improved and covered by tests

## Cutter

This year was the focus of improving user interface and user experience:

- Improve stability (fix various crashes)
- RzMark APIs (a new feature allowing selecting data and code ranges in both Rizin and Cutter)
- Keybindings search [shortcuts-pr](https://github.com/user-attachments/assets/e43d36ee-07a0-4e2a-a7ba-6f3ad5181caa)
- Add a scrollbar to the disassembly window
- Update disassembly color theme by "lightness"
- Use the new string search API from rizin
- Remember IO mode for recent files
- Fixed various translation issues

## Miscellaneous

[Google Summer of Code 2025](https://summerofcode.withgoogle.com/programs/2025/organizations/rizin)
