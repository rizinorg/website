---
author: "xvilka"
title: "Google Summer of Code 2021 Announcement"
date: "2021-05-19"
summary: "An announcement of the Google Summer of Code 2021. Three accepted candidates."
tags: ["rizin", "cutter", "gsoc"]
ShowToc: true
TocOpen: false
weight: 2
---

[Google Summer of Code 2021](https://rizin.re/gsoc/2021) is here and we are excited to participate! 🎉.  This is the 2nd internship program we are running this year, along with [RSoC 2021](https://rizin.re/posts/rsoc-2021-announcement).

We received many applications, and we are happy that there is a substantial interest in the project. We thank every participant
and wish them luck in their future endeavors. We also thank Google for providing us the platform for
attracting new contributors. Many of the past participants stayed with the project after GSoC,
and we sincerely hope it will continue in the future.

This summer, the accepted projects aim to improve the correctness and quality of the Rizin and Cutter
output, along with advancing user experience for embedded reverse engineering and exploitation.

## Aswin: Support for CPU and platform profiles

Hey, I'm [Aswin](https://github.com/officialcjunior)! I'm a sophomore year
undergraduate from India and I'm thrilled to be working with Rizin this summer.
For Google Summer of Code, I will be working on adding support for CPU and platform profiles.

I have always had a passion for reverse engineering as well as on how computers work at a low level. I hope to learn more about reverse engineering and hardware platforms by participating in GSoC at Rizin. I chose this as my project as it will help Rizin be more compatible with obscure hardware platforms, architectures and chips.

For the microtask, I was working on writing an SVD parser for Rizin. It was basically a plugin which lets you open SVD files inside Rizin and make use of all the information about the peripherals and registers present inside the file. I came to know about so many things about microcontrollers and many other things like Memory Mapped IO, registers and a lot more about platforms and on how they work while working on this. Over the summer, I'm going to do the following tasks:

* Get the configuration variables related to the CPU and the platform to be dynamically populated by listing the available dedicated SDB files and add them to the analysis loop.

* Provide `rz-lang` and `rz-pipe` bindings so that the users can choose through scripts, as well.

As an additional goal, I'll be working on improving the SVD parser and adding it to Rizin as a core plugin, so that it'll be shipped with Rizin and Cutter, as well. At the end, I also hope to write a good how-to article on how to add and use a profile

I am engrossed by the wonderful feeling of community I have felt during contributing to Rizin. I have gained amazing insights and skills and feel very grateful to be working and learning more from one of the smartest, kindest and knowledgeable people I've ever known.

Hoping to accomplish great things and have a really great summer. I wish the very best to all the folks who got in!

## Pulak: Heap viewer for Cutter

Hi, I am Pulak Malhotra from India. I am an undergraduate student and researcher at IIIT Hyderabad. GSoC provides me an excellent opportunity to work on real-world codebases, contribute to the open-source community, meet new people and learn new things. I am relatively new to reverse engineering. In the past, I enjoyed working on low-level systems in my university courses which drew me towards reverse engineering. Rizin's welcoming and helpful community is a significant factor that makes me want to contribute to the project.

My GSoC project aims to deliver widgets that provide information about the heap state while debugging programs in Cutter. These widgets will give information regarding chunks in a heap and the bins in which free chunks are located. I also aim to deliver graphical visualization of the linked list of the arena bins. My project also includes refactoring the heap codebase in Rizin, so the heap allocator is dynamically selected based on binary. Currently, the heap allocators are compiled at compile time in Rizin. I will also try to add support for more heap allocators in Rizin.

I contributed the Pull Request [#810](https://github.com/rizinorg/rizin/pull/810) as my microtask. I improved the output of existing heap-related commands like `dmh` and `dmhf` in this microtask, and I created new commands `dmhv` and `dmhd`. `dmhv` is similar to `dmh` command but provides extra information about the heap chunks and `dmhd` command provides concise information about the bins of main arena. At various points, while solving the issue, I felt lost, especially when I was not familiar with the codebase. To get over this, I would make small changes, rerun the code, and note the difference in output. At every step, many members of Rizin gave me detailed advice and feedback and guided me. I also came across some suggestions which I pursued further in PR [#912](https://github.com/rizinorg/rizin/pull/912) and issue [#1088](https://github.com/rizinorg/rizin/issues/1088).  Overall, it was a fantastic experience contributing to the Rizin.

## 08A: Refactoring ELF binaries loading

Hi, I'm [08A](https://github.com/08A) from France. I'm an undergraduate student at EPITA, majoring in Systems, Networks and Security (SRS).

Some friends of mine are Free Software gurus, and they motivated me to contribute to Open Source software. So I chose to contribute to a tool I already used.

I started working on the code base with Radare2, and after the fork I switched to Rizin. The majority of my time was allocated to fix various issues found by clang-analysis and refactoring how Rizin parses ELF information. The overall experience was like going down a rabbit hole, the code base is huge and some parts are rusty. But the community is awesome and I learned a lot of things.

For this summer of code, I will be going to work on refactoring the ELF loading feature. The main challenge will be to fix the imported function detection. If you want additional information, you can check this [link](https://summerofcode.withgoogle.com/projects/#5375712095109120).
