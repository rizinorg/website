---
author: "Rot127"
title: "Epistemology"
layout: "blog"
tags: ["rizin", "research", "ai", "binary analysis", "reverse engineering"]

ShowToc: true
TocOpen: false
---

# Literature and Experiments

**Introductory reading**

- Knowledge engineering: building cognitive assistants for evidence-based reasoning - Chapter 1.1 to 1.4 {{< citation KnowledgeEngineeringBuilding2016 "Chapter 1" >}}

# The Gap in Computing Reasoning

Most of the tools we use in reverse engineering (RE) of computer programs are from the 2000s and earlier.
Maybe they were implemented more recently, but the algorithms and ideas are relatively
(for computer science time spans) old.
The basics were always the same: a disassembler, some flow and type analysis, optionally
a decompiler and a debugger. And besides the note book and the mind of the researcher,
that is essentially it.

But gradually restoring knowledge over the software at hand is more than just applying tools.
The experience of the researcher, their systematic way of detangleing the information
and making sense of it is the core of the work.\
_So why is it that we don't have more tools which mimic it?_

The tools and algorithms we use do automation. They are of deductive nature,
computing from one set of known facts another set of facts.
They _necessarily_ need all premises to be true so their conclusions are true as well.
That is the case for a disassembler, which outputs garbage
if the wrong bytes are fed in, and good output for the correct ones.
It is also true for basic control flow analysis: if a `jump 0x7000` instruction is located in an `r-x` map,
it is pretty much guaranteed to jump to `0x7000` when executed.

While these deductive methods are a center piece in RE they are unable to provide results,
if one or more premises are unknown or uncertain but needed for a conclusion.
If our `jump 0x7000` is located in an `rwx` map, suddenly we can't be sure anymore that it actually jumps to that address.
Because at the time of execution the instruction could have been overwritten.
There simply is no guaranteed clear answer our premises infer.
We simply miss the premise "does anything write to the location where `jump 0x7000` is?".
And with it our deductive algorithm fails.

Of course, that doesn't stop us from inferring valid conclusions.
The process of inferring just gets more fuzzy from here on.

Researchers work with hypotheses, educated guesses, or choose the most likely result from their point of view.
That kind of reasoning is called abduction. Abduction no longer makes it _necessary_ that inferring
from true premises leads to a true conclusion.\
If we give evidence `E` and hypothesis `H1, H2, ..., Hn` to an abductive reasoning process,
it chooses hypothesis `Hi` which best explains `E`, or which seems closest to the truth.
{{<citation douvenAbduction2025 >}}
{{< footnote "The exact definition is part of the philosophical debate." >}}
This already helps to argue with a lack of information, but these methods also have problems.\
How do we know if `Hi` is the _best_ explanation for `E`?
What does _best_ even mean?\
And even if we somehow know that, what happens if all our hypothesis we have are a bad lot? {{<citation douvenAbduction2025 >}}\
It is easy to imagine a novice researcher looking at some evidence and struggling to come up with any useful hypothesis.
Simply because she misses experience.

And lastly, there are the inductive reasoning methods.
These are usually understood as being of enumerative or cumulative nature.
The more observations we collect as evidence, the more
likely our hypothesis is true or false. {{< footnote "Of course this doesn't exclude the case where a single observation can flip the whole conclusion. If we observe 1 million white swans, there is a high probability that all swans are white. Of course only until we observed a single black swan." >}}
{{< citation eagleProbabilityInductiveLogic2025 "Chapter 1.5">}}\
There are a few (often statistical) algorithms doing that.
Think of Value Set Analysis (VSA): {{<citation balakrishnanAnalyzingMemoryAccesses2004>}}
with every new observation the possible values of a memory location get more or less certain.\
In daily life we would simply call this reasoning "generalization from observations and experiences".
And of course inductive methods suffer from the same problem as generalization does:
It is simply not obvious which observations can count as evidence and which not. {{< citation eagleProbabilityInductiveLogic2025 "Chapter 1.5">}} {{<citation wikipediaRavenParadox2026>}}

I think we can all see how these modes of reasoning are applied in RE.
And sketching them out so explicitly it seems obvious that _most_ reasoning in RE is
either abductive or inductive.
Sure, it happens in the head and notebook of the researcher. But they are nontheless dominant!

As researchers we struggle by definition with the lack of information we are trying to regain.
And uncertain results can be exceptionally useful if the alternatives are
no results at all or if they are computationally unfeasible.
So we trade uncertainty for answers. We can grasp the bigger picture by letting go of
the strictness of deduction. Fuzzy reasoning can provide clarity and orientation in uncertainty.

And yet, here we are. The tools nowadays don't allow us to encode our experience in code.
It is too fuzzy.
They don't allow us to share intuition in a way that it is reusable by others.
The scripts and plugins almost always handle the specific case, not the general one we have in our head.
And by sharing the special case we share the deductive rules - which fail the moment a premise like a hardcoded offset changes.
We can't share the general case from our head so easily.
Because sharing it means we have to write it down in computable form.

_So why can't we make our fuzzy intuition and experience computable?_

The answer is quite simple in our understanding:
There are simply not that many tools which allow it.
Most tools are not built to work with answers like "maybe", "possibly", and "a likelihood of p".
Not many algorithms use sampling and statistics.
And even the ones which do rarely make the uncertainty transparent.
Let alone letting the researcher control the thresholds of it.

A huge part of RE is abductive and inductive, but there is no way to express it _computationally_ so it can be shared.
The non-deductive reasoning stays in the researcher's head.
For the lack of a method to preserve it.

Having a fuzzy understanding of the patterns in a binary, even if wrong in the details, brings value.
That intuition is a map helping with the exploration, finding the places one wants to look with the magnifying glass.

Expressing intuition in computable form enables us to share it.
Sharing our intuition means sharing our maps, so others can use them to find the places
quicker to use their magnifying glass.

# Reasoning Theories

## Want to join?

Here we can add intros for each reasoning theory.

If you want to contribute such an intro as well, feel free to open a draft PR
so we can discuss it.

The introduction should talk about the reasoning process's applicability to reverse engineering.
It should give examples and explain little experiments you did.

You _should not_ explain the reasoning process in detail again.
You, as the author, are required to read the primary sources, of course,
and understand its advantages and shortcomings.

The page should contain at least:
- A brief refresher and links to more detailed introductions about the reasoning theory.
- A description of applicability in reverse engineering.
- Examples showing how it can be used.

### Factor Graphs

Not necessarily a philosophy. But applicable for us.

### Bayesian Networks

### Bilattices {{<citation fittingBilatticesSemanticsLogic1991>}} {{<citation bakhtiariBilatticeLogicEpistemic2020>}} {{<citation shetPredicateLogicBased2011>}}

### The Stability Theory of Belief {{<citation leitgebStabilityTheoryBelief2014 >}}

### Additive Logic of Epistemic Reasons {{< citation leitgebAdditiveLogicEpistemic2026 >}}

### Fuzzy Logic

### Deep Learning

## Theory comparisons

- What are the limits of these theories.
- Is that how people actually reason? {{< citation douvenRoleExplanatoryConsiderations2015 >}}
- What can each of them do, what can they not?

{{< citation_list "Citations" >}}

{{< footnote_list "Footnotes" >}}
