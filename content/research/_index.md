---
author: "Rot127"
title: "Research"
layout: "blog"
url: "/research"
summary: "research"
tags: ["rizin", "research", "ai", "binary analysis", "reverse engineering"]

ShowToc: true
TocOpen: true
---

# Abstract

Most software reverse engineering tools on the market are limited to a standard set
of analysis algorithms and functionalities. A framework commonly consists of
disassemblers, an intermediate language to emulate processor instructions,
and a set of deductive algorithms for data and control flow analysis.\
Many of the applied techniques were already described in the 1990s and earlier.
Since then, computer science has made significant progress in the fields of
knowledge engineering, complex pattern recognition, and artificial intelligence,
often applying probabilistic and more "fuzzy" techniques to learn and make
predictions about complex systems.\
With a few notable exceptions, the lessons from these fields have not been applied
in reverse engineering yet, although they have already proved exceptionally valuable
in biology, medicine, intelligence, climate science, and other fields.\
Software reverse engineering at its core is the understanding of complex systems; nonetheless,
there is a gap between what theoretical computer science provides and what is
applied in practice.\
We want to change this. In this public research and development effort, we will
evaluate theoretical concepts for their applicability to reverse engineering,
develop prototypes, use them, and bring them to a production-ready state for
a wide variety of processor architectures.

# On Restoring Knowledge

To give you an idea of how lessons from the knowledge engineering field could be used,
consider the following two examples.

## Inductive Example: Classifying a string parser function

You open a binary and are interested in checking string parsers in it
because they are often a source of bugs.
What we want is that users can just type a prompt and
see a heat map of code regions.
The brighter certain spots are, the higher the likelihood they are
string-parsing code.

How could this be done?

---

Let's consider what indicators there are to infer that some code is
parsing a string:

- It could call functions that have something to do with regular expressions.
- It could contain loops breaking on a read `\0` byte.
- It could check read bytes if they are in the ASCII range.
- It could pass read bytes to `atoi`.
- It references strings that have the word "parser," "parsing error," or "parse" in them.
- Its memory access patterns could be
  - sequential (reading character by character, word by word)
  - or jump once, then read sequentially (reading an offset to a substring, parse the substring).
- And probably many more indicators...

Some of these indicators give stronger evidence than others.
For example, if code references a string that reads something like "parsing error at 0x%x,"
it is a strong indicator it parses something (not necessarily strings, though!).
The memory access patterns in the loops are maybe less significant.
But if we observe a "parsing error" message referenced in a loop that breaks on a `\0` byte,
we can be pretty certain.

Let's visualize how the indicators could impact an inferred certainty that the observed code
is indeed parsing strings.:

![Example of inferring from observations a new property](/images/example_string_parsing.svg)

The certainty of the indicators (mapped to a `[0,1]` interval
{{<footnote "Don't confuse this with probability. Probability theory obeys certain axioms. Using 'certainty' here is meant to be vague, so it simply conveys the idea instead of being too technical.">}})
can come from many different sources.
The `regex_match()` call is maybe statically known, and we are 100% sure the code calls it.
We also know that there is a loop that breaks on `\0`.
There was also some code _possibly_ checking for ASCII, but that is uncertain.
`atoi` was definitely not called, and while there were some strings referenced that were similar to "parser,"
there was no direct match. And lastly, the memory access pattern didn't really follow our requirements.

The result is 0.7, which can be plotted as a shade in a heat map, giving an idea
of how strong the "string parser" categorization is compared to other code regions.

It is important to understand that the certainties for these observations
can, in themselves, be a product of a similar reasoning structure.
For example, the `refs "parser" string` could come from a Jaro–Winkler distance,
`Checks ASCII` from symbolic execution, `Calls atoi` could be checked statically,
and `Loop breaks for \0` was maybe set manually by the user. {{<footnote "A.k.a information integration: https://en.wikipedia.org/wiki/Information_integration">}}
Of course, the network's rules don't need to use these naive computations of weighting and summing.
Factor graphs, Bayesian networks, Markov chains, or whatever you can implement in code are possible.

You quickly see that one can build pretty complex reasoning networks like that.
Changing their results the more observations happen.

Classification does not need to stop on this small scale.
Small observations of reasoning networks can accumulate.\
Where in the binary are potential read or write primitives? \
What is the factual architecture of the program, where are control flow clusters,
and where are the bridges between them?
What is the function of these regions of memory?

All questions we expect to answer.\
In the end, these are questions a reverse engineer answers.\
Except with scribbles in a notebook, so why not in computational form?

> If you want to see an actual implementation of this principle here
> check out FunProbe {{<citation kimFunProbeProbingFunctions2023>}}.
> It builds a bayesian network from different kind of hints to infer functions.

## Abductive Example: Enhancing LLM reasoning with rapid access to low level facts.

LLMs are already a huge help in generating hypotheses about artifacts in a binary.

"What is the purpose of this function?", "What could the data represent?",
"What should I do if I look for the bootloader code in this binary?"

LLM might hallucinate more or less in the answers.
Though, if the researcher is not familiar with the type of binary and struggles
to come up with new ideas, even a rough direction can help.

To improve this we want the LLM to generate an answer which is verifiable
by a proof system. So it can be validated and is less verbose than natural language.
Increasing the quality of the hypothesis the LLM can generate.

Consider the following function from a niche architecture (Hexagon).

```
; segment.modem.b02
┌ fcn.fe102000();
│           0xfe102000      ┌   immext(##0x80)
│           0xfe102004      └   R3:2 = combine(##0xad,#0x0)
│           0xfe102008      ┌   P0 = boundscheck(R1:0,R3:2):raw:lo
│       ┌─< 0xfe10200c      │   if (P0.new) jump:t 0xfe102030
│       │   0xfe102010      └   R2 = ##0x0
│       │   0xfe102014      [   R2 = ##0xce
│       │   0xfe102018      ┌   immext(##0xc0)
│       │   0xfe10201c      └   R3:2 = combine(##0xda,R2)
│       │   0xfe102020      ┌   P0 = boundscheck(R1:0,R3:2):raw:lo
│      ┌──< 0xfe102024      │   if (!P0.new) jump:nt 0xfe102050
│      ││   0xfe102028      └   R2 = ##0x4
│      ││   0xfe10202c      [   R0 = add(R0,##0xffffff32)
│      │└─> 0xfe102030      ┌   immext(##0xfe1082c0)
│      │    0xfe102034      │   R3 = memw(R1+##0xfe1082f4)
│      │┌─< 0xfe102038      └   if (cmp.eq(R3.new,#0x0)) jump:nt 0xfe102050
│      ││   0xfe10203c      [   R3 = add(R3,R1)
│      ││   0xfe102040      [   R3 = memw(R3+R2<<#0x2)
│      ││   0xfe102044      [   R3 = add(R3,R1)
│     ┌───< 0xfe102048      ┌   jump 0xfe102054
│     │││   0xfe10204c      └   R0 = memw(R3+R0<<#0x2)
│     │└└─> 0xfe102050      [   R0 = ##0xfffffbad
└     └───> 0xfe102054      [   jumpr LR
```
Assuming you have no decompiler, it will take a short while until you figure out
what it does.

An LLM will, possibly in a shorter time, provide you a few observations about it:
- It does one or two bounds checks (intervals `[0xad,0x00]` and `[0xda,0xce]`) on the input value in `R1:0`.
- It accesses a multi-level lookup table and returns data from it.
- Due to the bounds check, there are around ~60 and ~170 entries in the table to retrieve.
  Possibly two kinds of versions for whatever is stored in there (one for each interval).
- From the `segment.modem.b02` flag, it might infer that it is a
  system/HVM/interrupt call lookup or a constant/state getter function in an
  early boot procedure (at least it did in my experiment).

If the researcher is inexperienced, these points can already be helpful to look for further evidence to
reject or accept these hypotheses.
But it would be even better if the agent could test the hypotheses on its own.

An obvious solution is, of course, to provide the agent an MCP server to run commands in the analysis tool.
It could disassemble the locations where the function is called and track
the returned value.

But this has disadvantages:
- The LLM has to keep potentially a lot of text in its context.
  The more often the function is used, the faster the context grows.
- LLMs will always, by design even, hallucinate.
  This likelihood will go up if the input is not strongly represented in its training data (niche assembly might be such a case).
  Combined with an arbitrary context growth rate, the probability of degrading output will increase.
- To prevent overarching context usage, LLM agents could dispatch individual tasks to other agents.
  But with each passing of messages to another agent, mistakes in communication can
  quite literally happen.

It is like letting an LLM build a math proof.
It may be sufficient to let it run on its own, but it is almost certain it
will be faster and more precise if it is given a formal proof checker.
In cooperation with another agent, it will also be better to give the colleague
agent a piece of code it can run to _obtain_ `X`,
instead of a lengthy explanation of how to _calculate_ `X`.

So, instead of letting the LLM do its reasoning on simple text alone,
we can expect to get better results if it can formally verify hypotheses or evidence.

For example, it querying a tainting algorithm, checking if the function's return value
is used in an indirect call.
If it finds such a case, it supports the system/HVM call hypothesis.
If it doesn't, it can check the alternative hypothesis again.

Additionally, it could dispatch a job to another agent,
asking it to find vulnerabilities in this function.
That agent could query for the exact semantics of `P0 = boundscheck(R1:0,R3:2):raw:lo`,
learning that the bounds check is `R0 in (0xad,0x00]` instead of `R1:0 in [0xad,0x00]`.
It might also spot the fact that `R1` is added to the table addresses but is never bounds-checked-
a potential read or write primitive if `R1` were user-controlled.

Instead of answering with a vague, informal explanation of this finding,
the agent could store the formal proof in the knowledge base
and only return the proof ID and a short note that controlling `R1`
could lead to an out-of-bounds read or write.

Any agent who has access to the knowledge base can later query it again
to make use of that fact.

We can be almost certain that building formal tools for agents will yield
results, because it was already done before.\
LLM coding agents are relatively good because they can, at any time,
test their generated text against a formally correct machine (a compiler, an interpreter, a test suite).

But reverse engineering tools rarely provide such _formalized_ and _rule-based_ access and results.
{{<footnote "A notable exception is BAP's reasoning system Saluki: https://github.com/BinaryAnalysisPlatform/bap-plugins/tree/master/saluki">}}\
Note that an MCP server for command access is not sufficient!
The value added by the knowledge base and inference engine is to reduce the complexity of the task.
Storing results in an information-theoretically dense form (formalized, instead of natural language)
won't dilute its precision.

What we need to achieve is a sufficiently powerful query language, knowledge base, and inference system
to express hypotheses and proofs, accessible for LLMs and humans alike.

Then an LLM can rely on short queries and their formally correct truthfulness,
instead of endless generated walls of text.
Humans, on the other hand, can add the bits and pieces agents cannot yet comprehend.

> Want to see a similar application of an LLM in production?
> Check out Binary Ninja's Sidekick in the [Related Work section](#related-work)!

## Why Rizin is a Good Choice

We, the core team of RizinOrg, believe that Rizin is the fitting framework for
our research effort.

- We think maintainability is an essential target in software engineering.\
  Many tools are abandoned after proofing the concept.\
  That is not what we are aiming for.\
  We want deployable software for production.
- Rizin supports a wide variety of architectures and binary formats.
  Cutting edge ones like LoongArch, as well as niche or out of production ones.\
  We always plan, implement, and test new features against the oddities of these architectures.\
  So everything works in the uncommon case as well.\
  Because the world is not just x86 and ARM.
- Rizin is written in C, is designed modular, and very lightweight (compile time commonly under 3 minutes).
- Our intermediate language RzIL is made for reverse engineering and is mathematically well defined.\
  A hard requirement for knowledge engineering and formal prove systems.
- Rizin binds to any Swig-supported language.
  Making it perfect as library for your favourite tool or automated analysis.
- Rizin is open source, keeping it free for everyone while its
  license still allows commercial use.

## Roadmap of Research

The roadmap from initial research to prototypes and finally production-ready code
is intended to have three steps:

1. **Foundational Concepts**\
  \
  **Progress on:** [Epistemology](/research/epistemology)\
  \
  Knowledge engineering first and foremost requires a definition of what "knowledge", "reasoning", or "inferring" actually means.
  For that we will turn to knowledge engineering and analytical philosophy, inspecting suitable theories.
  To stay on topic, we will accompany every theory with practical reverse-engineering examples to show their workings and shortcomings.\
  Tools like Problog or Datalog should be sufficient for rapid prototyping of minimal examples.\
  The aim is to:
    - Understand the problem space.
    - Get an overview of what each theory provides, what limits they have, and whether those limits are relevant.
    - Build introductory articles for anybody who wants to join, providing reading lists and relevant examples.
  \
  The result should be a selection of reasoning theories to implement.
2. **Algorithm Design and Prototyping**\
  Once the theories are selected, we can think about the best possible implementation of them.
  The question is how a knowledge base, reasoning engine, language, and the human-machine interface
  should look in practice.
  This will include:
    - Testing existing tools and libraries and considering them as dependencies.
    - Researching efficient algorithms for the computational problems we have to solve.
    - Designing the UX/UI and API.
    - Prototyping and testing.
3. **Final Implementation**\
  Lastly, the lessons learned must be converted into a product.\
  Lessons learned from Step 2 are applied here by implementing the most suitable
  solutions.

## Related Work

- Binary Ninja has an LLM as [Sidekick](https://sidekick.binary.ninja/blog/sidekick-26-1-a-proper-home-for-sidekick/).
  It allows asking questions about the binary and produces summaries, findings, and artifacts.
- Binary Ninja supports experimental [semantic search](https://docs.sidekick.binary.ninja/guide/semantic_indexing.html).
  The search term can be a vague description ("TLS handshake"); then matching functions
  are returned. The user has control over the similarity (uncertainty) level of the matching.
- The Binary Analysis Platform (BAP) has a reasoning engine called [Saluki](https://github.com/BinaryAnalysisPlatform/bap-plugins/tree/master/saluki).
  It lets you define rules and tries to apply them against the binary.
  There are rules to match simple backdoors, user-controlled stack pointers, or unsanitized SQL handling.
- There is a growing research interest in using probabilistic reasoning to infer information about binaries:
  {{<citation kimFunProbeProbingFunctions2023>}}
  {{<citation zhangRevampingBinaryAnalysis2023>}}
  {{<citation peiXDAAccurateRobust2020>}}
  {{<citation yuDeepDiLearningRelational2022>}}
  {{<citation shinRecognizingFunctionsBinaries2015>}}
  {{<citation wangSemanticsAwareMachineLearning2017>}}
  {{<citation baoBYTEWEIGHTLearningRecognize2014>}}

# Want to Join?

If you or your organization is interested in contributing or joining the effort,
we are happy to get in touch!

You can contact us via:

- [Email](mailto:core@rizin.re)
- [Mattermost](https://im.rizin.re)

{{< citation_list "References" >}}

{{< footnote_list "Footnotes" >}}

