---
layout: post
date: 2019-07-29 12:00
categories: Week09
---

## REvision

* `REP` instruction (repeat). E.g. if in ecx (count), it will count down until it's 0.
  * often used in strcpy, initialising array to all 0's etc.
* `ESP` vs `EBP` offsets
  * ESP continually changes, EBP is constant for the entire function
* Structs
  * Offsets and the way variables are used (e.g. use of 1-byte registers imply char, dereferencing vs. not dereferencing).
  * Char would be mov byte (not mov dword) etc.
* Arrays
  * all operations on the array will be the same size (unlike structs)

_Tooling:_ `strace` (syscall tracing), `ltrace` (library call tracing)

## Shellcode
* `mprotect` to change memory region permissions
  * Jump to PLT (lookup) to call functions
  * Use rop to mprotect, then run your own shellcode
* Avoid ``'\0x0a'`` and `'\x00'`

**Partial Return Address Overwrite**
* ASLR only really randomises the middle of the address
  * All pages start at 000, and the upper byte is usually the same too.
* Since the number is stored in little endian, it's very easy to just overwrite the lower byte
  * And the offsets will be the same for the lower few bytes, so you can work out where functions are and jump to them (relative)

## Source Code Auditing
* Bad API Usage
  * With `strncpy` if you put exactly n characters, it won't put the null byte in.
* Heap
  * Use after free, double free etc.
  * Custom malloc implementations
* Integer overflow/underflow
  * look at how the number is later used and what this could do
* Type Conversions
* Incorrect use of `sizeof()` (returns size of pointer)
* Pointer arithmetic
  * Can crop up in for loops
* Forgetting to kill after checking permissions etc.
* Race conditions

## ROP
* Chaining functions
  * To ensure yo ucan call another function, you need a gadget such as `pop pop ret` to tidy up the stack after the function call.
    * CDECL: caller does housekeeping

**Stack Pivots**
```bash
$ ropper -f file --stack-pivot
```
* 'Stack Pivot instructions': changes ESP somehow
  * E.g. `xchng e?x, esp`, `add esp, x`, `ret x`
* Moving the stack frame
