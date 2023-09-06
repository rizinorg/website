---
author: "Billow"
title: "GSoC 2023 - Enhancing DWARF Support"
date: "2023-09-06"
summary: "In this article, I discuss my experience enhancing DWARF support in Rizin for Google Summer of Code 2023."
tags: ["rizin", "dwarf", "gsoc"]
ShowToc: false
weight: 2
---

Hi! I'm [Billow](https://github.com/imbillow), and I had the privilege of participating in GSoC 2023, working on improving DWARF support for the Rizin project. In this blog post, I'm excited to share my journey, the challenges I faced, and my future plans for this project. Let's dive right in!

Over the past few months, my primary focus has been on enhancing DWARF support within Rizin. DWARF (Debugging With Arbitrary Record Formats) is a crucial standard for debugging information in binary files. My work brings significant improvements, including the introduction of exprloc, compressed debug sections and composite variable storage.

To showcase some of my achievements, I'm comparing the disassembly output obtained using the `pdf` command for the `write_fmt<Write>` function in the ELF file [dwarf_rust_bubble ↗](https://github.com/rizinorg/rizin-testbins/raw/master/elf/dwarf_rust_bubble) before and after my DWARF contributions were integrated. The enhanced output demonstrates Rizin's improved ability to parse DWARF debugging information and precisely locate variables.

```
[0x00005180]> pdf @ dbg.write_fmt_Write
...
┌ dbg.write_fmt<Write>();
│           ; var int64_t var_28h @ stack - 0x28
│           ; var int64_t var_18h @ stack - 0x18
│           0x00010270      push  rbx                                  ; impls.rs:155 ; struct Result<(), std::io::error::Error> write_fmt<Write>(struct Box<Write> *self, struct Arguments fmt);
...
```

```
[0x00005180]> pdf @ dbg.write_fmt_Write
...
┌ struct Result<(), std::io::error::Error> write_fmt<Write>(struct Box<Write> *self, struct Arguments fmt)
...
│           ; arg struct Box<Write> *self @ rsi
│           ; arg struct Arguments fmt @ ...
│           0x00010270      push  rbx                                  ; impls.rs:155 ; struct Result<(), std::io::error::Error> write_fmt<Write>(struct Box<Write> *self, struct Arguments fmt)
...
```

Another example, I'm comparing the disassembly output obtained using the `pdf` command for the `iterPreorder` function in the ELF file [dwarf_go_tree ↗](https://github.com/rizinorg/rizin-testbins/raw/master/elf/dwarf_go_tree). The `iterPreorder` function shows arguments and local variables precisely located on the stack. Most notably, the tree argument is represented as a composite variable spread across multiple stack locations. This new composite storage capability handles complex DWARF types. Overall, the improved output matches the original DWARF debugging data much more closely, rather than using generic `unknown` types. This showcases Rizin’s significantly upgraded DWARF parsing including features like composite variables and the immense value delivered to reverse engineers through my work.

```
[0x0045d5a0]> pdf @ dbg.main.tree.iterPreorder
...
┌ dbg.main.tree.iterPreorder(unknown_t visit, unknown_t t);
...
│           ; arg unknown_t t @ stack + 0x10
│           ; arg unknown_t visit @ stack + 0x20
│           ; var unknown_t traverse @ stack + 0x40
│       ┌─> 0x00491ce0      mov   rcx, qword fs:[0xfffffffffffffff8]   ; tree.go:26 ; void main.tree.iterPreorder(struct main.tree t, func(int) visit);
│       ╎   0x00491ce9      cmp   rsp, qword [rcx + 0x10]
```

```
[0x0045d5a0]> pdf @ dbg.main.tree.iterPreorder
...
┌ void main.tree.iterPreorder(main.tree t, func(int) visit)
...
│           ; var func(int) traverse @ stack - 0x40
│           ; arg main.tree t @ composite: [(.0, 64): stack + 0x8, (.0, 64): stack + 0x10, (.0, 64): stack + 0x18]
│           ; arg func(int) visit @ stack + 0x20
│       ┌─> 0x00491ce0      mov   rcx, qword fs:[0xfffffffffffffff8]   ; tree.go:26 ; void main.tree.iterPreorder(main.tree t, func(int) visit)
```

While I'm proud of these accomplishments, the project did not come without its challenges. Working on a project of this magnitude required grappling with the complexity of the DWARF5 standard and ensuring compatibility across architectures and binary formats through rigorous testing. Additionally, collaborating remotely with the Rizin community was essential but posed communication and coordination challenges.

However, through dedication and guidance from my mentors, I was able to overcome these hurdles. Moreover, my journey with Rizin is far from over. Looking ahead, there are several exciting plans on the horizon:

- **Continued DWARF5 Improvements**: I will keep an eye on DWARF developments and ensure Rizin remains up-to-date with future revisions.

- **Performance Optimization**: There is always room for performance optimization. I will explore ways to make Rizin even more efficient when dealing with DWARF information. The main idea is that we can make DWARF load only when needed, instead of loading all DWARF directly.

- **Unifying Debug Information**: As the reverse engineering landscape continues to evolve, the need for unified support of various debuginfo formats like DWARF, PDB, and others becomes increasingly evident. In the spirit of unification, I am excited to take on the challenge of integrating and harmonizing these diverse debuginfo standards within Rizin. This ambitious endeavor aims to provide a seamless experience for developers and analysts working with different binary formats, making Rizin an even more versatile and indispensable tool in the field of reverse engineering. Stay tuned for updates on this exciting journey towards unified debuginfo support!

In conclusion, participating in GSoC 2023 has been an invaluable learning experience. I've expanded my skills, contributed to the Rizin project, and become part of a vibrant open-source community. There is still work to be done, but I'm excited about the future and making reverse engineering more efficient for everyone through enhancements like unified debuginfo support. Thank you to the Rizin community and my mentors for making this journey possible!
