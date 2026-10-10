---
author: "Rot127"
title: "Language"
layout: "single"
summary: "research"
tags: ["rizin", "research", "ai", "binary analysis", "reverse engineering"]

ShowToc: true
TocOpen: true
---

# Language

## Requirements

- We shouldn't invent it on our own: eats too many resources, adds maintenance burden.
- It shouldn't be too difficult to port existing recipes from BAP or Binja's BNQL.
- Query language should be both human-readable and machine-readable.
  - Clear (not too bloated) syntax.

## Language/Language Engine candidates to use

### [Wirelog](https://github.com/semantic-reasoning/wirelog/) - C implementation of a Datalog engine.
  - Currently active developed.
  - Part of the startup [Cleverplant](https://cleverplant.com).
  - Requires signing of [commercial](https://github.com/semantic-reasoning/wirelog/blob/main/CLA.md) license for contributions.

  **Misses**
  - [No probabilistic extension](https://github.com/semantic-reasoning/wirelog/issues/909)
  - [No support](https://github.com/semantic-reasoning/wirelog/blob/272edf3a24b25676f12c4b843d55510f5048dd2f/wirelog/wirelog-types.h#L183) for values wider than 64bits
