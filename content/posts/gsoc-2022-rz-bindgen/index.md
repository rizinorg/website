---
author: "wingdeans"
title: "GSoC 2022 - rz-bindgen"
date: "2022-10-01"
summary: "An overview of rz-bindgen's design, implementation, features, and future."
tags: ["rizin", "cutter", "gsoc"]
ShowToc: false
weight: 2
---

Hello. I'm [wingdeans](https://github.com/wingdeans/), a participant of GSoC 2022 with Rizin. For the past few months, I've been working on creating [rz-bindgen](https://github.com/rizinorg/rz-bindgen) - a framework for making Rizin scriptable from other languages.

This document covers some of the design decisions and internals of the tool.
To get started with the bindings, see [the usage documentation](https://github.com/rizinorg/rz-bindgen/blob/master/doc/README.md).

# Rationale
[Rz-pipe](https://github.com/rizinorg/rz-pipe/), the currently recommended way to script Rizin, only works with commands exposed to the Rizin shell. Although it can do everything the Rizin shell can, it cannot match the full Rizin C API in performance, feature-completeness, or type guarantees. The C API on the other hand, is more difficult to work with, especially for one-off scripts. Rz-bindgen seeks to be a middle-ground, making the C API accessible from other programming languages. Python is the primary target for rz-bindgen, as it is usable for both scripts and plugins, and has been incorporated successfully in other reverse-engineering tools.

# Design
Many languages already have tools for creating bindings to C/C++, such as [rust-bindgen](https://github.com/rust-lang/rust-bindgen) for Rust or [CLIF](https://github.com/google/clif) for Python. However, these tools often rely on mapping C++ constructs to their own, and require extra work to create idiomatic bindings for plain C code. Like many of these tools, rz-bindgen parses C headers and generates bindings as output. However, rz-bindgen targets one project and multiple languages, rather than one language and multiple projects. This allows rz-bindgen to make use of Rizin-specific annotations, such as the `RZ_NULLABLE` and `RZ_DEPRECATE` C macros.

See [this post on the Rizin blog](https://rizin.re/posts/gsoc-2022-announcement/#wingdeans-automated-python-bindings) for more details on the thought process behind my proposal and my implementation ideas from before I started the task.

# Implementation
I considered my primary options for parsing the C headers to be [tree-sitter](https://github.com/tree-sitter/tree-sitter/) and [libclang](https://clang.llvm.org/docs/Tooling.html#libclang). Even though I wrote about tree-sitter in the Rizin GSoC announcement blogpost, the integrated preprocessor and semantic analysis led me to choose [libclang's Python bindings](https://libclang.readthedocs.io/en/latest/).

## C Structs and Functions
Once a header is parsed, C data structures are grouped with functions that operate on them. In [this snippet from rz-bindgen](https://github.com/rizinorg/rz-bindgen/blob/e010ce8d688cfbb12e99dc15868d818aeda21b5b/src/bindings.py#L146-L170), the `RzAnalysis` struct from the `rz_analysis.h` header is grouped with functions that have the `rz_analysis_` prefix. In the generated Python bindings, these groupings are mapped to object-oriented classes, with the `RzAnalysis` class containing the grouped functions as its methods. The `RzAnalysis` class also makes all the fields of the C struct accessible except for `leaddrs` (which is ignored as per the `ignore_fields` argument) and `type_links` (which is renamed as per the `rename_fields` argument).
```python
rz_analysis = Class(
    analysis_h,
    typedef="RzAnalysis",
    ignore_fields={"leaddrs"},
    rename_fields={"type_links": "_type_links"},
)

rz_analysis.add_method("rz_analysis_reflines_get", rename="get_reflines")
rz_analysis.add_prefixed_methods("rz_analysis_")
rz_analysis.add_prefixed_funcs("rz_analysis_")
```

## Generation
Rz-bindgen is designed to support multiple backends to generate bindings for a variety of languages. A backend takes the `Class` objects created in the transformation step and generates output. There are, at the time of writing, a [SWIG](https://www.swig.org/) backend and a [Sphinx](https://www.sphinx-doc.org/) backend.

The SWIG backend is currently only used for Python bindings, but SWIG targets other languages, such as Java and OCaml. Supporting them in rz-bindgen should be relatively simple.
The Sphinx backend generates documentation for the Python bindings and can be viewed [here](https://wingdeans.github.io/rz-bindgen/classes/RzAnalysis.html).

## Generics
One of the main challenges in translating the C headers was the existence of generic container types. Rizin uses types like `RzList` and `RzVector` to represent a linked-list and dynamic array respectively and, being written in C, uses `void*` for the type of the data contained within. This means that trying to use these types from Python would be difficult, as their elements lack the type information to generate methods. Fortunately, Rizin developers were already annotating the types of these functions for developer ergonomics using comments such as `RzList /*<RzAnalysisBlock *>*/ *bbs`.

This allows bindings to use container types in a type-safe manner. In [this Python example from rz-bindgen](https://github.com/rizinorg/rz-bindgen/blob/9ccbc56cefaf043b95666793dd1bc156480bbe6c/examples/3-cle_bin_plugin.py#L47), a specialized `RzList_RzBinSymbol` is created, and `RzBinSymbol`s are appended to it. Appending any other type will result in an error.
``` python
syms = rizin.RzList_RzBinSymbol()

for sym in self.loader.main_object.symbols:
    binsym = rizin.RzBinSymbol()
    binsym.thisown = False
    binsym.name = sym.name
    binsym.type = rizin.RZ_BIN_TYPE_FUNC_STR
    binsym.paddr = sym.linked_addr
    binsym.vaddr = sym.rebased_addr
    binsym.size = sym.size
    syms.append(binsym)
```

# Additional Features
The snippet above is from an example of implementing an `RzBinPlugin` in Python. See [the bin_plugin documentation](https://github.com/rizinorg/rz-bindgen/blob/9ccbc56cefaf043b95666793dd1bc156480bbe6c/doc/bin_plugin.md) for more details.

The Python bindings also make it easier to access Rizin internals when writing scripts, as can be seen in the [rz_cmd example](https://github.com/rizinorg/rz-bindgen/blob/9ccbc56cefaf043b95666793dd1bc156480bbe6c/examples/4b-rz_cmd.py) (see [the cmd documentation](https://github.com/rizinorg/rz-bindgen/blob/9ccbc56cefaf043b95666793dd1bc156480bbe6c/doc/cmd.md) for more details). One key feature is the ability to register a Rizin command backed by a Python function, like so:
```py
def print_function_info(fn: rizin.RzAnalysisFunction):
    print("name:", fn.name)
    print("number of xrefs from:", len(fn.get_xrefs_from()))
    print("number of xrefs to:", len(fn.get_xrefs_to()))
    return True
core.register_group("u", "A custom group for user-defined commands")
core.register_command("uf", print_function_info)
```

The [Rizin plugin](https://github.com/rizinorg/rz-bindgen/blob/9ccbc56cefaf043b95666793dd1bc156480bbe6c/plugin) registers Python as an `RzLang`, allowing users to load Python scripts on the fly. It also adds add a `core` variable to the `rizin` Python module, allowing scripts that import it to access Rizin's own `RzCore`.

# Reflections
The coverage of the bindings is currently lacking - it is not yet possible to use every bit of the C API. I hope this will change as I get more eyes on the project.
I also hope to improve the Rizin plugin and finalize the [Cutter plugin](https://github.com/rizinorg/rz-bindgen/blob/9ccbc56cefaf043b95666793dd1bc156480bbe6c/cutter).

In the long term, I hope to add bindings for extensions such as [rz-ghidra](https://github.com/rizinorg/rz-ghidra) which expose their functions. This could allow access to Ghidra's P-Code and decompiler once implemented.

I would like to thank my GSoC mentors [XVilka](https://github.com/XVilka) and [megabeets](https://github.com/ITAYC0HEN), as well as Rizin core contributors [ret2libc](https://github.com/ret2libc) and [deroad](https://github.com/wargio).

If you need help with rz-bindgen or wish to build a project using the generated bindings, feel free to reach me on the [Rizin mattermost](https://rizin.re/community/) @wingdeans (we have an IRC bridge too).
