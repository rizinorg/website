---
author: "xvilka"
title: "Google Summer of Code 2022 Announcement"
date: "2022-05-23"
summary: "An announcement of the Google Summer of Code 2022. Two accepted candidates."
tags: ["rizin", "cutter", "gsoc"]
ShowToc: true
TocOpen: false
weight: 2
---

[Google Summer of Code 2022](https://rizin.re/gsoc/2022) is here and we are excited to participate! 🎉.  This is the 2nd year we participate in GSoC as a Rizin organization.

We received many applications, and we are happy that there is a substantial interest in the project. We thank every participant and wish them luck in their future endeavors. We also thank Google for providing us the platform for
attracting new contributors. Many of the past participants stayed with the project after previous GSoC and RSoC programs, and we sincerely hope it will continue in the future.

This summer, the [accepted projects](https://summerofcode.withgoogle.com/programs/2022/organizations/rizin) aim to improve the quality of analysis of Rizin by employing our next generation intermediate language - [RzIL](https://github.com/rizinorg/rizin/blob/dev/doc/rzil.md), along with making the scripting and automation easier with Rizin and Cutter by improving the API, especially the Python one.


# Dhruv: RzIL uplifting migration

Hey, I'm Dhruv Maroo ([DMaroo](https://github.com/DMaroo/))! I am a computer science and engineering undergraduate student from Indian Institute of Technology, Madras. I am excited to work on [RzIL migration](https://rizin.re/gsoc/2022/#rzil-uplifting-migration-350-hour-project) of x86 architecture as a contributor under GSoC 2022.

I have been interested in computer security ever since I started learning computer science and engineering. Out of all the various subdomains within security, binary exploitation and reverse engineering appealed to me the most. I was also fascinated with systems engineering and low-level programming. Recently, I had also been exposed to the relevance of intermediate languages in symbolic execution. Given these interests, working on RzIL under Rizin was a golden opportunity for me.

I started contributing to Rizin in August 2021. It took a fair bit of time to get to know the code base, but the maintainers were very helpful and supportive. Without their help, I wouldn't have accomplished this. Since August 2021, I have worked on a variety of issues and features, like project compression, seeking and autocompletion for global variables, breakpoint serialization, type pretty printing API, porting `db` (debug breakpoint), `c` (compare) and shell commands to newshell, fixing Coverity scan issues and memory leaks, RzIL refactoring, DWARF attribute type checking, and lots of other miscellaneous issues. I am also working on migrating the SuperH ISA to RzIL ([#2518](https://github.com/rizinorg/rizin/pull/2518)).

The RzIL uplifting project is a 350-hour project, spanning 5 months, starting from the second week of June. During the project, I plan to implement the 8086, 80186, 80286, 80386 and 80486 instructions from the x86 instruction set. Along with these, I will also be implementing Pentium and MMX instructions. This will be followed by testing the migration using [`rz-tracetest`](https://github.com/rizinorg/rz-tracetest) and getting the traces. I will also be adding API commands in Rizin to visualize and interface with the IL tree. These commands will be augmented with corresponding widgets in Cutter. Then, I am planning to document all of this in the [Rizin book](https://book.rizin.re/), to allow others to easily use it and contribute as well. One of the optional (but very interesting) goals is to use the power of RzIL in the binary analysis loop, to provide better analysis and new features. Other optional goals involve migrating SSE and AVX instructions to RzIL and migrating [other architectures](https://github.com/rizinorg/rizin/issues/2080) to RzIL as well.

I am looking forward to start working on the project and improve Rizin. I would again like to thank the maintainers to help me through my contributions, and I would like to thank GSoC for giving me such a great learning opportunity.

# wingdeans: Automated Python Bindings

Hello. I'm [wingdeans](https://github.com/wingdeans/), a second year Computer Science major at the University of Florida.

I'll be working on exposing the Rizin native API to Python, as an alternative to the current [rz-pipe command-based API](https://github.com/rizinorg/rz-pipe). This 175-hour project will involve semi-automatically generating Python bindings from the Rizin C headers, as well as abstractions and documentation to increase developer ergonomics.

#### Rationale
Python has seen widespread adoption among the infosec and reversing communities, with several reverse engineering platforms integrating Python as a scripting and plugin language. Although rz-pipe exposes all the Rizin commands to a number of languages, including Python, the integration is not as robust as Rizin's native C API.

The completion of this task will make scripting with Rizin more pleasant. It will also help Cutter, which supports Python GUI plugins.

#### Considerations
The bindings should feel Pythonic. At the bare minimum, this will involve creating classes for the various structs in Rizin, and creating methods out of any C functions that manipulate those structs. Python features like keywordargs should also be used when appropriate.

The bindings should be somewhat automated. This will ensure that the bindings do not become out of sync with the Rizin core API. In addition, many Rizin functions already contain annotations about nullability (`RZ_NULLABLE`/`RZ_NONNULL`) and ownership (`RZ_OWN`/`RZ_BORROW`). Parsing this information will be useful in managing memory.

#### Rough Plans
The generator will first parse the Rizin headers, perhaps using [tree-sitter](https://tree-sitter.github.io/tree-sitter/), and extract function information. It will then create classes and methods from a user-specified list of functions and emit source code to be integrated with additional manually-written modules.

#### About Me
I've been doing CTFs for about a year now, with a focus on cryptography and reversing. For reversing challenges, I primarily use Cutter, which was why I ended up applying here for this year's GSoC. I’m interested in enhancing Rizin’s scripting capabilities for use in custom reversing tools and plugins.

For my microtask, I'm implementing [support for dotnet binaries](https://github.com/rizinorg/rizin/pull/2528/files). Although the overall file format is the same as in PE files, there are additional headers, streams, and tables that need to be parsed. I also ended up writing a disassembler plugin, and an analysis plugin is in the works.

I look forward to working with the Rizin team to implement a better scripting API, and I hope these additions will encourage users to extend Rizin and Cutter for their own reversing needs.
