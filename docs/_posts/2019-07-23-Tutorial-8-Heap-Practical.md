---
layout: post
date: 2019-07-23 12:00
categories: Week08
---

## Tooling

**pwndbg vis_heap_chunks is broken** unless you specify the address and length (for wargame challenges - compilationerror or something)

## Notes for Challenges
* The vulnerability will be one of:
> * Double Free
> * Use after Free
> * Buffer Overflow
> * Use of uninitialised memory

* Random chunks that weren't allocated - probably something in libc using malloc
* Fast bins never update the 'prev in use' bit - because it doesn't do checks to merge. The purpose of this bit is to check whether it should merge
