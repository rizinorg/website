---
author: "xvilka"
title: "Google Summer of Code 2023 Announcement"
date: "2023-05-05"
summary: "An announcement of the Google Summer of Code 2023. Two accepted candidates."
tags: ["rizin", "gsoc"]
ShowToc: true
TocOpen: false
weight: 2
---

Like all previous years, we are grateful to Google for being able to participate in [Google Summer of Code 2023](https://rizin.re/gsoc/2023).
We received many applications, and we are happy that the project has substantial interest. We thank every participant and wish them luck in their future endeavors. We also thank Google for providing us with the platform for
attracting new contributors. Many of the past participants stayed with the project after previous GSoC and RSoC programs, and we sincerely hope it will continue.

This summer, the [accepted projects](https://summerofcode.withgoogle.com/programs/2023/organizations/rizin) aim to improve the quality of handling debugging information in Rizin and uplifting more architectures to our next-generation intermediate language - [RzIL](https://github.com/rizinorg/rizin/blob/dev/doc/rzil.md).

## billow: Debug information handling improvements

Hello. I’m [**billow**](https://github.com/imbillow). A silent open-source enthusiast. I will be working on improving the handling of debugging information in Rizin. 

I started contributing to Rizin in November 2021. It took some time to get to know the code base, but the maintainers were helpful and supportive. Without their help, I wouldn't have accomplished this.
Since November 2021, I have worked on various issues and features, like uplifting 8051 architecture to RzIL, refactoring graph-related code, porting the commands parser to the tree-sitter-based one, and fixing windows and cpp compatibility, EBCDIC character support. I am also working on uplifting TriCore architecture to RzIL ([#3478](https://github.com/rizinorg/rizin/pull/3478)).

The plan, based on the project description and tracking issue [#1285](https://github.com/rizinorg/rizin/issues/1285), involves several key objectives. Firstly, we will enable support for loading **DWARF information from separate files** [^1] and **debuginfod** [^2]. Next, we will unify access to source lines/types information for DWARF, PDB, dSYM, and refactor or fix any parsing code as needed. This will be followed by the integration of source line and types/variables information with the "p" commands in debug mode for seamless printing. Furthermore, we will integrate this information with breakpoint commands and APIs to enhance overall functionality. Finally, we will implement parsing performance improvements to optimize the project's efficiency.

[^1]: **DWARF** is the debuginfo standard used on most operating systems today. **Separate debuginfo files** are files that contain debugging information extracted from executable binaries and shared libraries. These files help developers debug and diagnose issues in their software without bloating the primary executable or library with debug symbols. Debuginfo files typically have a .debug extension and are generated using tools like 'objcopy' or 'eu-strip'. They enable a more apparent separation between production code and debugging data, resulting in smaller binaries and improved performance in production environments, while still providing necessary debugging information when needed.

[^2]: **Debuginfod** is a service that provides a convenient way to access debugging information for software components, such as executables, shared libraries, and separate debuginfo files. It is part of the ELFutils project and is designed to simplify the process of debugging and tracing software by automatically locating and retrieving the required debugging data over HTTP. Debuginfod works with various debugger tools like GDB, LLDB, and SystemTap, allowing developers to focus on debugging their code without worrying about managing debuginfo files.

## brightprogrammer: Uplifting MIPS to RzIL

Hi! I’m Siddharth Mishra (a.k.a brightprogrammer). I'm a student of Mathematics & Computing at the Birla Institute of Technology, Mesra. I like developing software in C/C++, and I love Reverse Engineering & Malware Analysis. My summer project this year is to [convert the MIPS assembly code to RzIL](https://rizin.re/gsoc/2023/#rzil-uplifting-migration-350-hour-project) (Rizin's Intermediate Language). This conversion will help improve Rizin's analysis module, which can help enhance reverse engineering efforts.

I started contributing around two months before the application submission deadline. I helped convert the old rizin shell to a new [tree-sitter](https://tree-sitter.github.io/tree-sitter/)-based [rizin shell](https://rizin.re/posts/rzshell/) for a command group named `cmd_help` ([PR#3421](https://github.com/rizinorg/rizin/pull/3421) and [PR#3452](https://github.com/rizinorg/rizin/pull/3452)). I've never had a formal course in Compilers and Formal Languages/Grammars, and this was the first time I saw how grammars are written! I was amazed by this. I also got a chance to write some tests for some changes I did. This was also a first-time experience. During this work period, I started to have a good bonding with my mentors and other awesome contributors/maintainers.

The uplifting process is divided into two parts :

 - The first part is to convert MIPS to RzIL.
 - The second part uses uplifted code to migrate analysis from ESIL (old intermediate language) to RzIL.

By improving analysis, I mean better function detection, type detection, structures' detection, better control flow graphs etc.

Other small tasks are involved, too, like adding support in Rizin to help visualize RzIL and to update/implement cutter widgets to use the new RzIL code. Each of these steps needs to be heavily tested by updating and using [`rz-tracetest`](https://github.com/rizinorg/rz-tracetest), which is used to generate instruction traces using [BAP's QEMU](https://github.com/BinaryAnalysisPlatform/qemu) and then compare the trace with RzIL's execution. If they match, and if we strongly assume that Qemu is emulating the code correctly, then RzIL is also working correctly!

I'm starting work early to complete the project on time. I want to work hard this summer and learn a lot. Besides, working on this project will improve my knowledge of IL which I need for a personal project that I intend to work on, which will act as a personal motivation for me.
