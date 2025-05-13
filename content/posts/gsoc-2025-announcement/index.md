---
author: "xvilka"
title: "Google Summer of Code 2025 Announcement"
date: "2025-05-13"
summary: "An announcement of the Google Summer of Code 2025. Three accepted candidates."
tags: ["rizin", "gsoc"]
ShowToc: true
TocOpen: false
weight: 2
---

 Once again Rizin project participates in [Google Summer of Code 2025](https://rizin.re/gsoc/2025). The program attracted numerous applicants, demonstrating significant interest in our project. We extend our sincere thanks to all applicants and wish them success in their future pursuits. We're also thankful to Google for creating this opportunity to connect with new contributors. It's worth noting that many former participants from previous GSoC and RSoC programs have become long-term contributors to our project, a trend we hope will continue.

This time we selected 3 projects, covering different aspects of the framework - from the analysis to debugging.

# Tushar Jain

Hey, I am Tushar Jain ([tushar3q34](https://github.com/tushar3q34)). I am a computer science undergraduate at Indian Institute of Technology Madras, India. I am excited to work on [improvement of analysis of object oriented languages](https://github.com/rizinorg/rizin/issues/416) and [devirtualization of method calls](https://github.com/rizinorg/rizin/issues/414) as a contributor in GSoC 2025.

I have a huge interest in reverse engineering and related cybersecurity topics. As a part of Cybersecurity Club in my university, I had been involved in giving CTFs, working on cybersecurity projects and learning more about low level programming for the past one year. I have been using Cutter to help me reverse engineer binaries for CTFs and Rizin and Cutter are tools that thousands of people use and depend upon. I am very interested in working on a project which will potentially improve the daily life of a lot of users worldwide.

## My past contributions

I got involved with contributing to Rizin in September 2024. It took some time to get used to the codebase but the maintainers were very helpful and supportive and their warming response to my queries was a big reason why I like contributing to Rizin. I have previously worked on a few improvements to Rizin like Refactoring Native Debugger, Optimizing `RzBuffer` on hotpaths and Fixing `aav` in MIPS and other architectures. I have also worked on small additions like Adding Linker Version to `rz-bin -I`, Fixing `rz-bin -P` segfault among other contributions. I am also currently working on [Migrating from Capstone to Zydis for x86 architecture](https://github.com/rizinorg/rizin/pull/4832).

## Abstract

As of now, the support for reverse engineering of Object Oriented Languages has been limited in the project. Currently, Rizin supports limited analysis of vtables and partial analysis of class heirarchy in C++ binaries. Similar analysis is largely absent for other major and widely used languages.


Since modern systems have a heavy usage of languages like C++, Rust, Java, ObjectiveC, Swift and many more, it is evident that Rizin should support efficient and accurate reversing of the specialized structures (classes, vtables etc) in these languages.

## Rough Plan & Targets

The project will mainly focus on five languages. A verbose version of the project description can be found in the [project proposal](https://docs.google.com/document/d/1A3tJaLfAewzNLIsBPR0V47bquw-vrkgq-Z8AhmM-uZ4/edit?usp=sharing) which explains the targets and the plan in detail.

**Analysis of C++ :**

- Improve analysis in binaries with or without RTTI with detailed graph visualisation
- Handle double dispatches and nested virtual functions
- Add visibility of object data-type to disassembly
- Connect methods with vtables in visual mode

**Analysis of Swift & Objective C :**

- Add `rz-libswift` demangler to rizin
- Add analysis of dynamic dispatch (vtables) in Swift
- Add analysis of message dispatch in Swift and ObjectiveC

**Analysis of Rust :**

- Analyse vtables for `trait` in Rust
- Analyse more complicated structures with dynamic dispatch

**Analysis of Java :**

- Keep track of `astore_<n>` in bytecode
- Add class hierarchy information


I look forward to working with the Rizin team and my mentors over the summer to improve the analysis of these languages which makes Rizin and Cutter better reverse engineering tools and helps users worldwide.

# Taksh

Hello! I am Taksh (aka [well-mannered-goat](https://github.com/well-mannered-goat)), an undergraduate student at IIITM Gwalior. I recently became interested in systems programming and bumped into Rizin while exploring software written in C. Through Rizin, I was exposed to reverse engineering and low-level programming and found my new interest in it. It also helped me rejuvenate my love for assembly code.

I started contributing to Rizin around December last year, beginning with basic issues like working on [filtering the output of the La command](https://github.com/rizinorg/rizin/pull/4786). The maintainers were really helpful in guiding me to set up the development environment and navigate the codebase.

Since then, I've worked on:
- Adding commands to show CPU descriptions ([PR#4847](https://github.com/rizinorg/rizin/pull/4847) and [PR#4861](https://github.com/rizinorg/rizin/pull/4861))
- Adding Python [3.11](https://github.com/rizinorg/rizin/pull/4976), [3.12 and 3.13](https://github.com/rizinorg/rizin/pull/5003) support in the PYC plugin of Rizin

The PYC plugin work was particularly interesting as I worked directly with pyc files and learned how Python code is compiled in versions above 3.11.0. I also gained insights into how disassemblers are written through this work.

## Project 

My project this summer involves rewriting the debugger support in Rizin. I would divide the project into two main parts:

### Target Platforms and Architectures
- **Linux**: x86, x64, PPC and PPC64 support
- **BSD Family**: FreeBSD, OpenBSD, NetBSD with x86, x64 and AArch64 support

### Implementation Strategy
The Travis CI of Rizin would be useful for testing different platforms, and QEMU would be used where direct hardware access isn't available. Following the maintainers' advice, I'll start by writing various tests for different platforms, this would help me learn the architecture-specific details necessary for debugger development.

I'll focus on the x86 and x64 architectures initially since they have abundant resources and documentation available.

## Optional Goals
If time permits, I'd like to extend the project to include:
- Adding memory leak and crash analysis tools
- Supporting additional OS and architecture combinations as mentioned in [PR#3644](https://github.com/rizinorg/rizin/pull/3644)

Memory leaks and segmentation faults are particularly interesting areas for me, and I would like to integrate some analysis capabilities for them in this project.

I'm looking forward to a great summer learning about debuggers on various platforms. Thanks to the maintainers and Google for this Google Summer of Code opportunity!

# Ahmed Kamal

Hi, I’m [Ahmed Kamal](https://github.com/ahmed-kamal2004) (@ahmed-kamal on Mattermost).
I’m a Computer Engineering student at Cairo University, a DevOps/Infrastructure engineer, and most importantly, a passionate contributor to the foundational software blocks that power the tech industry.

My journey with RizinOrg began with PR [#5031](https://github.com/rizinorg/rizin/pull/5031), followed by PR [#5056](https://github.com/rizinorg/rizin/pull/5056). Through this experience, I found Rizin’s community to be vibrant, welcoming, and deeply committed to advancing reverse engineering tooling for the security space.

For Google Summer of Code 2025, I’ll be working on making Rizin thread-safe, this project aims to improve thread safety within the Rizin reverse engineering framework by eliminating global static variables and replacing them with per-instance context structures, which will require a deep dive into Rizin’s codebase.

Basically, the project will include a set of tasks, including:

-  RzUtil (`sys.c`, `path.c`): Replace static variables with RzSys and RzPath context structures and remove constructor/destructor functions.
- RzCore (`agraph.c`): Encapsulate static state into a non-exposed AGraphContext.
- libmagic: Update library and clean up of any static variables.
- RzIO Plugins: Create a dedicated context per plugin to manage plugin-specific state.
- RzCons (`cons.c`, `output.c`): Carefully refactor static state into non-exposed structures while preserving Windows-specific behaviors "by testing on Windows environment".
- RzCore Terminal UI (TUI): Encapsulate static state into a context to support multi-threaded TUI operation.
- SDB: Remove unused static-variable-dependent functions, and refactor necessary ones to use context structures.

By the end of the program, Rizin will be thread-safe and ready to support multi-threaded, asynchronous processing, paving the way for seamless integrations with other tools like Cutter Desktop.

I’m excited to contribute to Rizin’s future and help enhance its performance, stability, and extensibility for the reverse engineering and security community.
