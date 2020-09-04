---
layout: post
date: 2019-07-30 12:00
categories: Week09
---

## Reversing other languages
### C++
* Functions have really stupid names
* Still similar to C, with prelogue and epilogue pushing/popping etc.
* Still have `get_pc_thunk`

**Objects**
* `Geeks::Geeks`  
  * Class called 'Geeks', containing a funtion called 'Geeks'
* _this_ operator
  * Pointer to a struct that has:
    * A collection of variables and function pointers
    * This is the current object, and the struct defines the functions/vars of this object
  * Usually some register is selected, and will be used with very strange offsets
    * This register is probably `this`
    * Just assume this is an argument to a function when reversing back into C
* _Vectors_
  * Arrays on the heap (that can be expanded using realloc)

### Go
* Many of the functions will be error handling etc.
* Try to ignore all the confusing stuff, and just identify the function calls
  * Try to work out the contents by looking for global variables and function calls

## Partial Overwrites
Function returning
```
 -------------------
|   [small buffer]  | < esp
|                   |
|     saved ebp     | < ebp
|        ret        |
|-------------------|
|  prev func stack  |
|           frame   |
|                   |
|   [big buffer]    |
|                   |
|                   |
|     saved ebp     |
|        ret        |
 -------------------
```

* `leave` instruction:
  * `mov esp, ebp; pop ebp;`
* If we overflow the small buffer one byte into saved ebp, then esp will move into our big buffer
* Now esp is pointing to our big buffer, so when that function ends, we will return to gadgets in big buffer
