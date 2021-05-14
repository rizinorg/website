---
author: "xvilka"
title: "Rizin Summer of Code 2021 Announcement"
date: "2021-05-13"
summary: "An announcement of the Rizin Summer of Code 2021. Two accepted candidates."
tags: ["rizin", "rsoc"]
ShowToc: true
TocOpen: false
weight: 2
---

We are excited to announce RSoC 2021! Rizin Summer of Code is a summer
internship program we organize together with  [KeenLab of Tencent](https://keenlab.tencent.com/zh/2021/03/04/RSoC-keenlab-2021/). We provide an opportunity
for students to work full-time on Rizin and RzGhidra.
We use the experience we gathered by participating in Google Summer of Code as an organization and
organizing our own RSoC as part of the radare2 project.

The application period continued through all of April with fierce competition between candidates. Finally, we chose
two students. Let them introduce themselves, and we wish them luck!

## Heersin: Intermediate language improvements

Hello, I'm [Heersin](https://github.com/Heersin) from China, an undergraduate student majoring in information security. At the very first, I was looking for a handful RE tool on the Linux platform, and then I met radare2. I have been using it to solve some basic CTF tasks and dumps from some malwares. I found there are some imperfect features (the concept of project, ESIL, search...) and want to contribute to it. After knowing there is a new fork named Rizin, which aimed at refactoring radare2, I started to get involved.

During my spare time, I've done some work for rizin, including:
   1. Update some out-of-date pages in the rizin documentation and add more examples.
   2. Fix some bugs in `pyc` plugin
   3. Add support for `luac` format
   4. Extend the testsuite to cover more platforms

For RSoC this year, I will be working on the ESIL and follow the [issue #277](https://github.com/rizinorg/rizin/issues/277) to refactor it, and will add support for floating point and bitvectors. I will also try to fix some issues in ESIL (e.g. prevent rizin from getting stuck when hitting some C-library functions).

It will be an exciting and challenging summer, looking forward to it!

## Basstorm: Types analysis

Hi, I'm [basstorm](https://github.com/Basstorm) from China, and I am a 21 years old student. Over the last couple of weeks, I have done some bug fixes and improved the class analysis module:

- Fixed display of duplicate vtables in `acll` command when using `aaa` command to analyze over 2 times.
- Improved the output of the `acll` command to be more concise and clear.
- Implemented the integration of data from the **RzBin** and **RzAnalysis** modules, which makes the results of class analysis more accurate.
- Implemented constructor and destructor detection based on the function name.

This summer, I am going to do the following tasks:

- Improve the support of PDB Structure.
- Implementing new features in **RzTypes**.
- Continue to implement new features or bug fixes around class analysis.
