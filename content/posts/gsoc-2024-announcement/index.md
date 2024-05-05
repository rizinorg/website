---
author: "xvilka"
title: "Google Summer of Code 2024 Announcement"
date: "2024-05-05"
summary: "An announcement of the Google Summer of Code 2024. Two accepted candidates."
tags: ["rizin", "gsoc"]
ShowToc: true
TocOpen: false
weight: 2
---

We are grateful to Google for being able to participate in [Google Summer of Code 2024](https://rizin.re/gsoc/2024).
We received many applications and are happy that the project has substantial interest. We thank every participant and wish them luck in their future endeavors. We also thank Google for providing us with the platform to
attract new contributors. Many of the past participants stayed with the project after previous GSoC and RSoC programs, and we sincerely hope it will continue.

This summer, the [accepted projects](https://summerofcode.withgoogle.com/programs/2024/organizations/rizin) aim to improve the ROP gadget searching capability, add an ROP compiler in Rizin, and uplift more architectures to our next-generation intermediate language - [RzIL](https://github.com/rizinorg/rizin/blob/dev/doc/rzil.md).

## z3phyr: Exploitation Capabilities Improvements

Hi! I’m [Giridhar Prasath Rajendran](https://github.com/giridharprasath) (a.k.a z3phyr). I’m a student pursuing my Master's in Cybersecurity at the University of Maryland, College Park. I have experience developing user-space networks and web applications. I enjoy playing binary exploitation CTF challenges and am passionate about anything low-level.

I started contributing to Rizin in January 2024 by fixing a UI issue [PR#4095](https://github.com/rizinorg/rizin/pull/4095). As a part of my microtask, fixing [Issue#1259](https://github.com/rizinorg/rizin/issues/1259) seemed perfect as I learned more about Linux heap exploitation. [PR#4355](https://github.com/rizinorg/rizin/pull/4355), [PR#4426](https://github.com/rizinorg/rizin/pull/4426) were merged as a part of fixing this issue. This helped me strengthen my knowledge about TLS and Glibc heap internals. I was also using this feature in my binary exploitation class assignments.

I will integrate the [ROP chain generation](https://summerofcode.withgoogle.com/programs/2024/projects/csgaDyzq) feature with the `rz-gg` tool. This involves:

- Gadget Analysis: Implementing functionality to analyze raw assembly gadgets, categorize them based on their semantics (e.g., load, store, syscall), and store this information in a Gadget DB.
- Gadget Selection and Chaining: Developing an API to process constraints (e.g., set register RDI to "/bin/sh", set RSI to NULL) using an SMT solver. This API will automatically select appropriate gadgets from the database to construct a functional ROP chain.
- Interaction: Provide an interface in `rz-gg` tool to allow users to generate ROP chains based on the specified constraints


The initial support will cover architectures like x86, x86-64, and ARM.

I look forward to a great summer of contributing and learning. I would like to thank the maintainers for their support and Google for providing this opportunity.

## moste00: Uplifting RISC-V Instructions to RzIL

Hello, my name is Mostafa Mahmoud (aka [moste00](https://github.com/moste00) @ github, kotlinenjoyer @ mattermost). I graduated from Cairo University, Faculty of Engineering, a Computer Engineer. I'm passionate about anything and everything involving metaprogramming, macro systems, compilers, developer tools, IDEs, debuggers, virtualization  & virtual machines, operating systems, computer architectures and hardware, plugin systems, and many other things! If I were to summarize my interests in one sentence, I would say that I like programs that "manage" or "provide services for" other programs, programs that serve as the bottom layers in a software stack, programs that manipulate other programs as data, and programs that other programs "run on top of", so to speak.

I started to contribute to Rizin in early March 2024. My first PR was [a cleaning up of and removing dead code in the source for the database SDB](https://github.com/rizinorg/rizin/pull/4328), then I cleaned up the code for the Z80 assembler by [removing all global variables and moving them into a state struct](https://github.com/rizinorg/rizin/pull/4343). The great people at Rizin were consistently helpful at every step, first providing tips on how to build Rizin, set up its dev env, navigate its large codebase, and provide helpful code reviews when I had the PRs ready.

On advice from my mentor in Rizin, I started to write a RzIL lifter for the very simple architecture MSP430, the purpose being to get used to writing IL lifters and encountering the challenges that people typically face while writing one. The incomplete lifter is [here](https://github.com/rizinorg/rizin/pull/4379). It still needs to be finished as of the time of writing this, but I hope it will soon be! 

The MSP430 pull request is itself just a micro-task and preparation for the main task I will do in GSoC 2024: Lifting the RISC-V architecture assembly into the RzIL intermediate language, a 350-hour project. I have set the MVP for this project at the point where the RV-32I and the RV-32F (Integer instructions and Floating-Point instructions, respectively) subsets of RISC-V are both lifted and tested using trace-testing, but see my [proposal](https://docs.google.com/document/d/1e__G28P2V8t3FKG_ZbJPDCFHfo2lMHMbvWeRl3zJ-0A/edit?usp=sharing) for stretch goals as well as more details, such as the proposed road plan.

In conclusion, by the end of my GSoC project in approximately October or early November of 2024, I hope that Rizin will have the capability to transform into RzIL the assembly for two new architectures, the MSP430, and the RISC-V instruction set. My goal is to implement this in a clean, efficient, and easy-to-reason-about and maintain fashion. I hope that I get more competent and knowledgeable about different Computer architectures and IL compilers because of working with all the smart and knowledgeable people around me in Rizin. I hope Rizin continues to attract contributors interested in low-level programming and reverse-engineering dev tools.

It's always a pleasure to participate in GSOC; I look forward to a summer full of hacking :'). 
