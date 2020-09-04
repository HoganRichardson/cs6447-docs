---
layout: post
categories: Week05
---
## Motivation

* Much easier to find bugs when reading C code (not assembly)
* Need to understand the purpose of the program in order to find the bugs

## Classifications of Bugs

* Bad API usage
* Logic Bugs
* Integer overflows
	* Most real-world buffer overflows are caused by overflow in buff size calculations
* Type ~~Conversion~~ Confusion (can lead to overflow)
* Operators 
* Pointer Aithmetic
* Race conditions
* Heap Overflows:
	* Use After Free
	* Failed malloc not checked (can allocate null, so that it gets dereferenced and used!)
	* Double Frees 

Many of these are down to programmer error

### Dangerous Functions
* `gets()`
* `printf()` when misused
* `strcpy()`
* `strncpy()` - doesn't add null terminator
* `memset()` - people mix up the arguments!