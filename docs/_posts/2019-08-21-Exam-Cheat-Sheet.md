---
layout: post
categories: Revision
---

## Exploitation
[Linux Syscall Reference](http://syscalls.kernelgrok.com/)

### Shellcode

**Launch /bin/sh** (no null bytes)
`25 bytes`

```nasm
xor eax, eax
push eax
push 0x68732f2f    ; //sh
push 0x6e69622f    ; /bin
mov ebx, esp
push eax

push SYS_execve
pop eax
xor ecx, ecx
xor edx, edx

int 0x80
```
<br>
**Read/Write to FD**
`56 bytes`
```nasm
; create 16 byte buffer on stack
push 0x61616161
push 0x61616161
push 0x61616161
push 0x61616161

; Read
mov eax, 0x03
mov ebx, 1000 ; fd
mov ecx, esp  ; buf
mov edx, 20   ; count

int 0x80

; Write
mov eax, 0x04
mov ebx, 1   
mov edx, 20  

int 0x80
```
### ROP
**Write Primative**

At some point, you may need to write to somewhere in memory, and then reference this address (e.g. if you need a pointer to a string).
1. Find a `rw` memory segment (`objdump -t binary | grep bss`)
2. Find a gadget of the form `mov [e?x], e?x; ret;`
    * This gadget can be used to write data from second register into the address pointer to in the first register

### Heap
**Exploit Checklist**

- [ ] Use After Free
- [ ] Double Free
- [ ] Heap Buffer Overflow
- [ ] Use of Uninitialised Memory

**Controlling Malloc Behaviour**
* `malloc_hook` and `free_hook` are function pointers that can be changed at runtime (i.e. if you want to use your own malloc implementation)

```c
/* __malloc_hook */
void *function (size_t size, const void *caller)

/* __free_hook */
void function (void *ptr, const void *caller)
```
<br>

#### Heap Data Structures
```c
struct malloc_chunk {

  INTERNAL_SIZE_T      mchunk_prev_size;  /* Size of previous chunk (if free).  */
  INTERNAL_SIZE_T      mchunk_size;       /* Size in bytes, including overhead. */

  struct malloc_chunk* fd;                /* double links -- used only if free. */
  struct malloc_chunk* bk;

  /* Only used for large blocks: pointer to next larger size.  */
  struct malloc_chunk* fd_nextsize;       /* double links -- used only if free. */
  struct malloc_chunk* bk_nextsize;
};
```
<br>
_Visualisation:_
* Note: the 'Red' region is the data chunk. When the chunk is allocated, the data will overwrite this region

![]({{ site.baseurl }}/assets/images/chunk-freed-CS.png)

<small>Source: [Azeria Labs](https://azeria-labs.com/heap-exploitation-part-2-glibc-heap-free-bins/)</small>

--------------------------------------------------------------------------------
## Reverse Engineering
### Data and Control Patterns
**Arrays & Structs**
* The size of offset constants and register usage can provide information about the size of the data structures in the array/struct.
* Structs are usually accessed via an offset into the struct (e.g. `[eax+4h]`). The first item in the struct will just be at the address of the struct

**Modular Arithmetic**

_Calculating the modulo from the magic number:_

1. m = 2<sup>32</sup> / magic number
1. If the line `sar edx, arg` exists, multiply m by 2<sup>arg</sup>

_Examples:_

| Modulo | Assembly | 2<sup>32</sup> / Magic Number
| --- | --- | --- |
| 6 | ![]({{ site.baseurl }}/assets/images/modulo/6.png) | 5.9999...
| 7 | ![]({{ site.baseurl }}/assets/images/modulo/7.png) | 1.7499...
| 9 | ![]({{ site.baseurl }}/assets/images/modulo/9.png) | 4.4999...

### x86 Architecture
#### Registers
![]({{ site.baseurl }}/assets/images/x86-registers.png)
<small>Source: [flint.cs.yale.edu/cs421/papers/x86-asm/asm.html](http://flint.cs.yale.edu/cs421/papers/x86-asm/asm.html)</small>

#### Flags
_Subset of relevant flags_

| Flag | Purpose |
| --- | --- |
| `CF` | Carry |
| `PF` | Parity |
| `ZF` | Zero |
| `SF` | Sign |

#### Instructions
_Frequent, Obscure and confusing instructions_

| Instruction | Purpose |
| --- | --- |
| [`cmp`](https://c9x.me/x86/html/file_module_x86_id_35.html) a1, a2 | Result = a2 - a1 (so if equal, zero flag set)|
| [`test`](https://c9x.me/x86/html/file_module_x86_id_315.html) a1, a2 | a1 AND a2 (so if equal, result is NOT zero - zero flag clear). Commonly used to test whether a value is zero (since `test a, a` sets ZF iff a = 0)|
| [`imul`](https://c9x.me/x86/html/file_module_x86_id_138.html) a1 | _Signed Multiply_. With one operand, multiplies a1 with the value in EAX. Result is stored in EDX:EAX. |
| `lea` & `mov` with `[]` dereferencing | The square brackets are equivalent to dereferencing a pointer (i.e. `*`). So `mov eax, [ebx]` will move the value pointed to by the address ebx into eax). Common arithmetic trick: `lea eax, [ebx*4]` will move the result of the ebx*4 expression into eax. |
| [`sar`](https://c9x.me/x86/html/file_module_x86_id_285.html) a1 | Part of the 'Shift' family. _Shift Arithmetic Right_. SAR sets the most significant bit to correspond to the sign bit of the original value. |

--------------------------------------------------------------------------------
## Source Code Auditing
### Bug Classification
* Bad API Usage
    * Incorrect arguments
        * E.g. mixing up arguments for `memset()` resulting in writing to 0 memory locations!
    * Use of Dangerous Functions
        * `gets()` is just bad (buffer overflow)
        * `strcpy()` and `strcat()` are just bad (buffer overflow)
        * `strncpy()` does not add a null terminator
        * `printf()` format strings
* Logic Bugs
    * Paths in the program that lead to privileged access of some kind that were unintended
* Data Types and Type ~Conversion~ Confusion
    * Integer Overflows
    * Implicit Type Conversions resulting in overflow/underflow
    * Pointer Arithmetic
* Race Conditions
* Heap Vulnerabilities:
    * Use After Free
    * Malloc return value not checked (allocate/use null!)
    * Double Frees
