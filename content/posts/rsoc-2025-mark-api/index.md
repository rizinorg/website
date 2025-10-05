---
author: "PremadeS"
title: "RSoC 2025 - Adding Mark API"
date: "2025-10-04"
summary: ""
tags: ["rizin", "rsoc", "marks"]
ShowToc: true
TocOpen: false
weight: 2
---

Greetings! I'm Emad Sohail (aka PremadeS), a 2nd-year Computer Science student at Information Technology University - Lahore. I'm passionate about contributing to open source projects mainly geared toward cybersecurity. You can find me on [Github](https://github.com/PremadeS) and [LinkedIn](https://www.linkedin.com/in/emad-sohail-130b3b265/).

In this project I introduced a new **Mark** API to support address-range annotations (marks) in Rizin, particularly for visual feedback in hexdump views. It can be thought of as an enhancement to the existing flag system. You can now mark entire ranges of addresses instead of tagging just one offset.

Below is a basic usage example for **Marks**

```
rizin /usr/bin/ls
 -- Enhance your graphs by increasing the size of the block and graph.depth eval variable.
[0x00006760]> m foo 0x6800
[0x00006760]> m+ bar 0x6945 this is another mark
[0x00006760]> ml
[0x00006760 - 0x00006800] foo
[0x00006760 - 0x00006945] bar
[0x00006760]> mC bar
this is another mark
```

You can explore the visual implementation of Marks in Cutter [here](https://cutter.re/integrating-the-rizin-mark-api-rsoc)

## What are Marks

A mark represents a user-defined range of addresses in the binary. Each mark can have:

* `From/To`: the starting and ending address (inclusive).
* `Name`: a unique identifier (escaped internally to avoid shell issues).
* `Real Name`: the original name of the mark as provided by the user.
* `Comment`: an optional note attached to the mark.
* `Color`: a color tag for visual identification in Cutter.

See the implementation of `RzMarkItem` below:

```c
typedef struct rz_mark_item_t {
	ut64 from; ///< inclusive starting address of mark
	ut64 to; ///< inclusive ending address of mark
	char *name; ///< unique name for each mark, escaped to avoid issues with rizin shell
	char *realname; ///< real name, without any escaping
	char *comment; ///< item comment
	char *color; ///< item color
} RzMarkItem;
```

## How Marks Work

The idea is simple: let users mark a range of addresses instead of a single offset. Each mark has a unique name and can also include a comment for extra context.

Marks are saved and loaded with the `.rzdb` project files so they are preserved just like flags.

Currently Marks are only shown in the hexdump view.

```
[0x00006760]> pxc
- offset -   0 1  2 3  4 5  6 7  8 9  A B  C D  E F  0123456789ABCDEF  comment
0x00006760  31ed 4989 d15e 4889 e248 83e4 f050 5445  1.I..^H..H...PTE  ; [s] foo [0x6760-0x6800]
0x00006770  31c0 31c9 488d 3d85 e4ff ffff 1527 0802  1.1.H.=......'..
0x00006780  00f4 662e 0f1f 8400 0000 0000 0f1f 4000  ..f...........@.
0x00006790  488d 3de9 0a02 0048 8d05 e20a 0200 4839  H.=....H......H9
0x000067a0  f874 1548 8b05 0608 0200 4885 c074 09ff  .t.H......H..t..
0x000067b0  e00f 1f80 0000 0000 c30f 1f80 0000 0000  ................
0x000067c0  488d 3db9 0a02 0048 8d35 b20a 0200 4829  H.=....H.5....H)
0x000067d0  fe48 89f0 48c1 ee3f 48c1 f803 4801 c648  .H..H..?H...H..H
0x000067e0  d1fe 7414 488b 0505 0802 0048 85c0 7408  ..t.H......H..t.
0x000067f0  ffe0 660f 1f44 0000 c30f 1f80 0000 0000  ..f..D..........
0x00006800  f30f 1efa 803d bd0a 0200 0075 2b55 4883  .....=.....u+UH.  ; [e] foo [0x6760-0x6800]
0x00006810  3de2 0702 0000 4889 e574 0c48 8b3d e607  =.....H..t.H.=..
0x00006820  0200 e8e1 deff ffe8 64ff ffff c605 950a  ........d.......
0x00006830  0200 015d c30f 1f00 c30f 1f80 0000 0000  ...]............
0x00006840  f30f 1efa e977 ffff ff66 2e0f 1f84 0000  .....w...f......  ; entry.init0
0x00006850  0000 0066 2e0f 1f84 0000 0000 0066 2e0f  ...f.........f..
```

## Why This Feature Is Useful

Sometimes you want to visually mark the start and end of a function, loop, or data structure in memory. A single flag isn’t enough to represent the whole span.

When analyzing malware or packed binaries, it’s useful to mark the section of memory that was unpacked or injected for quick reference.

Having a separate abstraction (Mark) helps decouple “range annotations” from “single-address flags” and avoids overloading one system with too many concerns.

It also lays the foundation to visually show the highlighted address regions in **Cutter**.

## Implementation

### Core modules & structure 

`rz_mark.h` defines the public API: structures (`RzMarkItem`, `RzMark`, etc.) and function signatures for creating, querying, editing, removing, iterating, and serializing marks. 

`mark.c` is where the bulk of logic lives: implementing the functions from the header, maintaining internal data structures, and integrating with command-line printing. 

`serialize_mark.c` handles serialization and deserialization of marks via `Sdb` (Rizin’s internal database), so marks persist across sessions.

`cmd_mark.c` and `cmark.c` handle the command-line functionality of marks.

`db/cmd/cmd_mark` contains **integration** tests, while `unit/test_marks.c` and `unit/test_serialize_marks.c` contain **unit** tests.

### Data structures & internal storage

At its core, a mark is represented by the `RzMarkItem` structure (explained above).

The container for all marks is the `RzMark` structure. It maintains:

- A **hash table** `(ht_name)` mapping names to items for fast lookups.
- A **list of items** `(items)` to store all marks in memory.
- A **skiplist** `(by_off)` to efficiently query marks by offset.

```c
typedef struct rz_mark_t {
	HtSP *ht_name; ///< name -> RzMarkItem*
	RzList /*<RzMarkItem *>*/ *items; ///< All marks
	RzSkipList *by_off; ///< offset -> RzMarksAtOffset*
} RzMark;
```

### Behavior & API implementation

#### Creation & Deletion

`rz_mark_new()` creates a new mark container.
`rz_mark_free()` frees the mark container.

#### Adding & Querying Marks

- `rz_mark_set()` defines a new mark within a given address range.
- `rz_mark_get()` looks up marks by name.
- `rz_mark_get_start()` gets a mark starting at a specific address.
- `rz_mark_get_end()` gets a mark ending at a specific address.
- `rz_mark_get_at()` gets a mark covering a specific address.
- `rz_mark_all_list()` retrieves all marks in the container.
- `rz_mark_get_all_off()` retrieves all marks at a specific offset.

#### Editing
- `rz_mark_rename()` renames individual marks.
- `rz_mark_item_set_comment()` annotates individual marks with comments.
- `rz_mark_item_set_realname()` sets the real name.
- `rz_mark_item_set_color()` visually tags with colors (for Cutter).

#### Removal
- `rz_mark_unset()` deletes individual marks.
- `rz_mark_unset_all_off()` clears all marks at a specific offset.
- `rz_mark_unset_glob()` removes marks in bulk using glob patterns.
- `rz_mark_unset_all()` removes all marks.

#### Iteration
- `rz_mark_foreach()` iterates through all marks.
- `rz_mark_foreach_glob()` iterates through marks filtered by glob patterns.

#### Saving/Loading
- `rz_serialize_mark_save()` saves marks to the Rizin database (`Sdb`).
- `rz_serialize_mark_load()` restores marks from the Rizin database (`Sdb`).

---

By exposing this API, Rizin ensures that marks are no longer limited to a single widget but can instead serve as a general-purpose annotation mechanism. This makes them accessible to plugins, scripts, and future Cutter features.

Refer to [PR #5313](https://github.com/rizinorg/rizin/pull/5313) for more details about the implementation of **Marks**.

## Work Done

Here is a brief overview of the work.

Added `rz_mark.h` / `rz_mark.c` with APIs for creating, querying, and serializing marks.

Implemented `m` commands to allow users to create and list marks.
Below is a list of supported commands for marks that can be brought up inside Rizin with `m?`:

```
[0x00006760]> m?
Usage: m[?]   # Manage marks
│m <name> <end> [<comment>] # Adds a mark if there are no existing marks.
│m+ <name> <end> [<comment>] # Add mark.
│m- [<regex_pattern>]    # Remove mark.
│m-*                     # Remove all marks.
│ml[j*qt]                # List all marks.
│ml.[j*qt]               # List all marks containing the current offset.
│mc <name> [<color>]     # Set a color for the given mark / Show the color for the given mark.
│mC <name> [<comment>]   # Set a comment for the given mark. If no comment is given, it shows the current one for the mark.
│mr <old> <new>          # Rename mark.
│mN <name> [<realname>]  # Show the realname of the mark / Set the realname of the mark.
│mm <name> <new_start> <new_end> # Move a mark to a new location.
│mf <regex_pattern>      # Distance in bytes to reach the starting of mark from the current offset.
│md[jt]                  # Describe mark.
│mi[j*qt] [<size>]       # Show marks in the current block or in the next <size> bytes.
```
Show start/end markers in comments section of hexdump (e.g. `[s] name`, `[e] name`).

Added tests under `test/db/cmd/cmd_mark` to validate command behavior.

Added `serialize_mark.c` to save and load marks with Rizin project files.

## Challenges

Although the process was relatively straightforward, the one major decision that had to be made was how distinct marks should be relative to existing flags. Some reviewers suggested combining the new API with flags; others argued for separation. The resolution leaned toward separation in the end.

## Future Work

Currently, marks are displayed in the hexdump using simple `[s] name` and `[e] name` tags to indicate the start and end of a range. While this works, it’s not always easy to see the full extent of a marked region at a glance. A future improvement would be to introduce visual bracket-style markers that span across lines, making ranges more obvious. *(See example below)*

```
0x00000100  ffff ffff ffff ffff ffff ffff ffff ffff  ................  ┐ [s] foo [0x100-0x120]
0x00000110  ffff ffff ffff ffff ffff ffff ffff ffff  ................  │
0x00000120  ffff ffff ffff ffff ffff ffff ffff ffff  ................  │┐ [s] bar [0x110-0x160]
0x00000130  ffff ffff ffff ffff ffff ffff ffff ffff  ................  ││
0x00000140  ffff ffff ffff ffff ffff ffff ffff ffff  ................  ┘│ [e] foo [0x100-0x120] 
0x00000150  ffff ffff ffff ffff ffff ffff ffff ffff  ................   │
0x00000160  ffff ffff ffff ffff ffff ffff ffff ffff  ................   ┘ [e] bar [0x110-0x160]
0x00000170  ffff ffff ffff ffff ffff ffff ffff ffff  ................
0x00000180  ffff ffff ffff ffff ffff ffff ffff ffff  ................
0x00000190  ffff ffff ffff ffff ffff ffff ffff ffff  ................
0x000001a0  ffff ffff ffff ffff ffff ffff ffff ffff  ................
0x000001b0  ffff ffff ffff ffff ffff ffff ffff ffff  ................
0x000001c0  ffff ffff ffff ffff ffff ffff ffff ffff  ................
0x000001d0  ffff ffff ffff ffff ffff ffff ffff ffff  ................
0x000001e0  ffff ffff ffff ffff ffff ffff ffff ffff  ................
0x000001f0  ffff ffff ffff ffff ffff ffff ffff ffff  ................
```

Extend the mark rendering to other views (e.g. disassembly, graph views) beyond hexdump.

## Conclusion

At the end, I would like to say that this RSoC has been a great learning experience. From not knowing anything about the Rizin codebase to adding an actual useful feature was definitely a great journey.

Special thanks to [@xvilka](https://github.com/notxvilka) for this amazing opportunity and also [@deroad](https://github.com/wargio), [@Rot127](https://github.com/Rot127) for the help and guidance.
