---
author: "ret2libc"
title: "Rizin shell"
date: "2021-12-31"
summary: "Rizin shell"
tags: ["dev", "rizin"]
ShowToc: false
weight: 2
---

Rizin is an interactive command line tool and as such it provides a nice shell, where you can execute rizin-specific commands to perform all kinds of actions like analyzing functions, getting information from a binary, showing sections and symbols, and much more.

It has a lot of commands that you must know in order to use it properly and for this reason we believe its shell should be as powerful and as discoverable as possible. In this post we are going to talk a bit about its basics and what we have done in Rizin to improve even further, for both users and developers (skip the Background section if you just care about the latter!).


# Background

Since the creation of radare2, the framework Rizin originally emerged from, more and more commands were added to the list of supported actions, help messages were written to help users navigate those commands and various constructs were developed to make the language recognized by the shell more and more powerful (and complicated, at times!). It is possible to execute a command and temporarily switch the architecture defined in Rizin, temporarily seek to another address in the address space, iterate over defined sections, functions or symbols, create macro, aliases and much more.

Commands are organized in a tree, where each letter represents a node in that tree. For example, `a` has several sub-commands like `aa`, `ab`, `af`, `ah`, `ao`, `av`, etc. `af` groups yet other sub-commands, like `afr`, `af+`, `af-`, `afl`, `afv`, etc. You get the idea. This tree can be explored by users thanks to the `?` suffix, which is used as a way to get help about commands.

```
[0x00000000]> afvb?
Usage: afvb   [idx] [name] ([type])
| afvb                        list base pointer based arguments, locals
| afvb*                       same as afvb but in r2 commands
| afvb [idx] [name] ([type])  define base pointer based arguments, locals
| afvbj                       return list of base pointer based arguments, locals in JSON format
| afvb- [name]                delete argument/locals at the given name
| afvbg [idx] [addr]          define var get reference
| afvbs [idx] [addr]          define var set reference
```

`?` can be used alone to get the sub-commands of the root node.

```
[0x00000000]> ?
Usage: [.][times][cmd][~grep][@[@iter]addr!size][|>pipe] ; ...
Append '?' to any char command to get detailed help
Prefix with number to repeat command N times (f.ex: 3x)
| %var=value              alias for 'env' command
| *[?] off[=[0x]value]    pointer read/write data/values (see ?v, wx, wv)
| (macro arg0 arg1)       manage scripting macros
| .[?] [-|(m)|f|!sh|cmd]  Define macro or load r2, cparse or rlang file
| ,[?] [/jhr]             create a dummy table import from file and query it to filter/sort
| _[?]                    Print last output
| =[?] [cmd]              send/listen for remote commands (rap://, raps://, udp://, http://, &lt;fd>)
[output truncated]
| t[?]                    types, noreturn, signatures, C parser and more
| T[?] [-] [num|msg]      Text log utility (used to chat, sync, log, ...)
| u[?]                    uname/undo seek/write
| v                       panels mode
| V                       visual mode (Vv = func/var anal, VV = graph mode, ...)
| w[?] [str]              multiple write operations
| x[?] [len]              alias for 'px' (print hexadecimal)
| y[?] [len] [[[@]addr    Yank/paste bytes from/to memory
```

Usually the letter of the group is representative of what the sub-commands do. So `p` stands for "print" and `pd` stands for "print disassembly" and so on. After some time, it becomes much easier to navigate this structure.

As mentioned above, you can create _statements_ that in one way or another modify the behaviour of a command (or multiple ones). Suppose you are analyzing a x86-32bit binary, but you know that some pieces of code will actually be executed in 64bit mode, you could do:

```
[0x00006b60]> s 0x6c00
[0x00006c00]> e asm.bits=64
[0x00006c00]> pd 2
            ;-- entry.fini0:
            0x00006c00      f30f1efa       endbr64
            0x00006c04      803d75b60100.  cmp byte [section..bss], 0  ; [0x22280:1]=0
[0x00006c00]> e asm.bits=32
[0x00006c00]> s 0x6b60
```

Or you can simply apply some modifiers to the `pd` command with `pd 2 @b:64 @0x6c00`. This statement will temporarily switch `asm.bits` and the current seek to execute `pd 2` within the right context. There are also other kinds of statements that allow you to redirect the output/error of a command to a file, or that provides a foreach-like behaviour. For example, if you want to execute a command on all segments, you could do `<cmd> @@iSS`. Iterating over all basic blocks of all functions recognized by Rizin could be done with `<cmd> @@b @@F`. See `@?` and `@@?` for more info.


# Improvements in Rizin shell

We rewrote from scratch most of the code dealing with parsing and handling of commands, moving from a simple and scattered approach to a more centralized and uniform one. We started this effort in radare2 with what was called `cfg.newshell` which since then became the default shell of Rizin and has improved even further. Some of the issues we had with the previous implementation and that led to the rewrite of this code: manually written command help strings inconsistently displayed, hand-written parser with a non clear formal and global grammar defined, impossibility of dynamically registering/deregistering commands in the shell, commands handlers code mixing core logic with input handling, manually implemented autocompletion of commands and partial autocompletion of arguments. These are just some of the things we solved.

Rizin keeps track of all commands that can be executed in its shell, their place in the commands tree, the brief summary of what they do, a possibly longer description, additional details that may be useful, the number of arguments a command accepts, their types and whether they are optional.

This information (and more) are stored by Rizin, making it possible to write code in a uniform way. For example, the list of sub-commands available under `z?` is automatically generated from the data Rizin has available. Autocompletion of commands can be easily performed as soon as a developer adds a new command. Argument autocompletion is straightforward as well, because when a developer adds a new command with its arguments (and types, which are mandatory), Rizin already has all the information to perform the autocompletion.

Also error reporting is more uniform, because when the shell detects a non-existing command it can immediately report that to the user. It is also possible to extend this behaviour to propose similar commands that a user wanted to type or provide help for the most similar command available. Similar errors can be reported when a user uses a command in the wrong way, for example by not providing enough arguments or by providing too many. Rizin, knowing how many arguments a command accepts, can immediately return an error (and this behaviour could be extended as well to provide the help of the command, to make life easier for users).

Another important aspect of having a database of commands which are available to the Rizin shell is that commands can be easily registered and deregistered at runtime by Rizin plugins (e.g. rz-ghidra). Of course you want to see the commands provided by a plugin only when that plugin is actually loaded. You also want your new command, defined by an external plugin, to behave similarly to the internal commands: you want the plugin commands and their arguments to be autocompleted and you want uniform error reporting. All this is possible because these operations are all automatically performed by the Rizin shell and not by each individual command.

We also recognize that sometimes having just a short summary of what a command does is not enough. You would want to have a much longer description of what operations the command performs, which config variables affect its behaviour, etc. This is why we added the possibility to provide a description to each command, that can be shown by using the `??` suffix. 

```
[0x00000000]> wv??
Usage: wv <value>   # Write value as 4-bytes/8-bytes based on value
Write the number passed as argument at the current offset as a 4 - bytes value or 8 - bytes value if the input is bigger than UT32_MAX, respecting the cfg.bigendian variable
Examples:
| wv 0xdeadbeef # Write the value 0xdeadbeef at current offset
| wv2 0xdead    # Write the word 0xdead at current offset
| wv1 0xde      # Write the byte 0xde at current offset
[0x00000000]> 
```

Only a few commands have such description right now, but we very much welcome pull requests to improve our user documentation. If you want to help us, see the next section to know which files to touch.

We also changed the way commands and statements are parsed: instead of relying on a very simple hand-written parser, we switched to a [tree-sitter](https://tree-sitter.github.io/tree-sitter/) based parser, where we just have to write a formal grammar and tree-sitter automatically generates the parser for us. This approach ensures that commands and their arguments are parsed in a consistent way. For example, all new commands accept quoted strings. Wrapping multiple words in single or double quotes would make the shell consider those words as a single argument for the command. If you need to pass `;` as an argument of a command (semi-colons usually represent the separator between commands), you can just quote it or escape it with `\;` and expect this to work for all (new) commands. Using tree-sitter and defining the grammar in a single file forced us to think about the grammar as a whole in a very uniform way across all commands and not just as the union of different grammars for each command.


# How it works

File [`rz_cmd.h`](https://github.com/rizinorg/rizin/blob/dev/librz/include/rz_cmd.h) has all the API to register, deregister, execute commands with a list of arguments and get help for a tree of commands. To add new commands, developers (of a plugin, for example) have to use `rz_cmd_desc_argv_new` by specifying the parent group in the commands tree, the handler of the command and a structure of type `RzCmdDescHelp` that describes the command: its summary, an optional longer description, a list of detailed sub-sections in the help and the list of arguments the command accept, including information on their types and whether they are optional or not.

As soon as the command is registered (e.g. a Core plugin with a command is loaded), the Rizin shell becomes aware of the new command: the command can now be executed, it is shown in the help tree in the right place, it is autocompleted as necessary, including its arguments. As an example, you can see [rz-ghidra](https://github.com/rizinorg/rz-ghidra/blob/32b82ed1d60755ad74433c706ca1a54ee7389dab/src/core_ghidra.cpp#L672) or [jsdec](https://github.com/rizinorg/jsdec/blob/master/p/core_pdd.c#L269).

Plugin developers have to use all the C API and data structures mentioned above and more, included in the `rz_cmd.h` file. To avoid a lot of boilerplate code and make changes to commands easier also for non developers we thought about auto-generating the C structures like `RzCmdDescHelp` from a list of [YAML files](https://github.com/rizinorg/rizin/tree/dev/librz/core/cmd_descs). Commands are all described in YAML files that mimics the final tree structure, like below:

```
  - name: tc
    summary: List loaded types in C format
    subcommands:
      - name: tc
        cname: type_list_c
        summary: List loaded types in C format with newlines
        args:
          - name: type
            type: RZ_CMD_ARG_TYPE_ANY_TYPE
            optional: true
      - name: tcc
        summary: Manage calling convention types
        subcommands:
          - name: tcc
            cname: type_cc_list
            summary: List all calling conventions
            modes:
              - RZ_OUTPUT_MODE_STANDARD
              - RZ_OUTPUT_MODE_LONG
              - RZ_OUTPUT_MODE_SDB
              - RZ_OUTPUT_MODE_RIZIN
              - RZ_OUTPUT_MODE_JSON
            args:
              - name: type
                type: RZ_CMD_ARG_TYPE_STRING
                optional: true
          - name: tcc-
            cname: type_cc_del
            summary: Remove the calling convention
            args:
              - name: type
                type: RZ_CMD_ARG_TYPE_STRING
```

While building Rizin, these YAML files are used to automatically generate a [.c](https://github.com/rizinorg/rizin/blob/dev/librz/core/cmd_descs/cmd_descs.c) and a [.h](https://github.com/rizinorg/rizin/blob/dev/librz/core/cmd_descs/cmd_descs.h) file containing all the data structures and C API calls necessary to construct the commands tree as described by the developer.

This approach ensures that commands shown in the help are only those that can be executed and that commands that can be executed are listed in the help as well. In the past, due to help messages being just strings manually written for each command, it was too easy to forget to update an help message and ending with a hidden command or with a wrong help that referenced a command which did not exist anymore.

Another big change for developers is that commands are not implemented anymore in [huge switch-cases](https://github.com/rizinorg/rizin/blob/6a0938574698cc80ed43bbb9594af06bb8fc4a10/librz/core/cmd/cmd_analysis.c#L1680) like before, but each [command handler](https://github.com/rizinorg/rizin/blob/6a0938574698cc80ed43bbb9594af06bb8fc4a10/librz/core/cmd/cmd_analysis.c#L7827) has its own function with a signature similar to the main function of a C program, including argc/argv arguments. We believe this makes our codebase much cleaner and easier to understand, with short (and less indented) command handlers that just have to deal with the core logic of the command, without having to add [boilerplate code](https://github.com/rizinorg/rizin/blob/6a0938574698cc80ed43bbb9594af06bb8fc4a10/librz/core/cmd/cmd_analysis.c#L4461) just to parse/split arguments like it was done before.


# Conclusion

We think we are making big changes towards a more usable, discoverable and descriptive shell and although these changes required a lot of time, we have reached a good point. Rizin is now in a mixed state, with some commands still following the old behaviour and other commands being switched to the new way described in this blog post. We are porting new commands approximately every week, but any help is appreciated: you can provide more accurate and descriptive summaries/description to the already converted commands in [https://github.com/rizinorg/rizin/tree/dev/librz/core/cmd_descs](https://github.com/rizinorg/rizin/tree/dev/librz/core/cmd_descs) or you can help us port commands following the old structure to the new approach, so that they can benefit of everything that is explained here (look at [#1342](https://github.com/rizinorg/rizin/issues/1342) to know which commands are missing). Have also a look at [rzshell.md](https://github.com/rizinorg/rizin/blob/dev/doc/rzshell.md) to know more about the shell.

If you have issues, bugs, ideas or want to discuss this approach or others with us, feel free to join us on [Mattermost](https://im.rizin.re/).
