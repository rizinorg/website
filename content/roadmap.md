 Concrete high-level feature areas and changes.

# 0.9

* Add support for missing members of H8 MCU family, and implement RzIL uplifting of them
* Complete migration from ESIL to RzIL for all supported architectures and features
* Improve FreeBSD, NetBSD, and OpenBSD debugging
* Improve ARM64 and PowerPC debugging
* Migrate from Capstone to Zydis for x86 architecture to address long-standing problems with unsupported x86 instructions
* Support STABS (pre-DWARF) debug information loading
* Add support for proper preprocessor in the type parser
* Refactor types to introduce type scope
* Rewrite RzNum to support proper formulas, bitvectors, floats, and so on
* Remove concept of the "block" in favor of direct transparent IO access

Full milestone is at https://github.com/rizinorg/rizin/milestone/21

# 1.0

* Add KB (Knowledge Base) support for storing metainformation in logic fact-based form
* Stable and documented API
* Refactor and merge various visual modes
* Refactor native debugger
* Big files loading support
* Remove GPL-only code in favor of LGPL
* Create documentation for the framework structure and all modules
* Create RzIL specification

Full milestone is at https://github.com/rizinorg/rizin/milestone/5
