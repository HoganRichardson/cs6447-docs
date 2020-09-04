---
layout: post
categories: Week07
---

## Return Oriented Programming
### Using Buffer Overflow to call 2 functions
```
 ----------------
|                |
|                |
|                |
|                | f2 arg2
|   f1 arg 2     | f2 arg 1
|   f1 arg 1     | Expected return for f2
|return addr f1()| <- manipulate to be f2
|address of f1() | <- original return address of overflow function,
|AAAAAAAAAAAAAAAA|                 now overwritten with address of f()
|AAAAAAAAAAAAAAAA|
|AAAAAAAAAAAAAAAA|
|      ...       |
|                |
 ----------------

```

* We can't do it and preserve arguments

#### Where can we redirect execution (when other protections in place)?
* Static Libraries
* Lib
* Pops the last item off the stack and jumps to eip

### ROP Gadgets
* Find a few instructions followed by ret somewhere in an executable region

E.g.

```asm
pop
pop
ret
```

* Some instuctions can be found by changing the offset on bytes (so it will treat the bytes differently).
	* In GDB x/12i 0xdeadbeef will display memory interpreted as assembly
* We can then jump to this to perform desired actions (e.g. to push the appropriate arguments onto the stack to call two functions at once)

#### Finding Gadgets
* Pwntools
	```python
	code = ELF('./mybinary')
	gadget = code.search(asm())
	```
* `ROPgadget`
* `ROPPER`

N.B. Don't use automated tools blindly to generate payloads! Use them to find instructions

### RET to Libc
* In some programs (e.g. small programs), it's hard to find the instructions you need.
* `libc` must be somewhere on the machine, so all libraries are there (we just don't know where they are)
  * we can use gadgets within libc in exactly the same way as using gadgets within the program
* _Where_ is libc? (in memory - what's the offset etc.)
  * GOT table: leak libc address.
  * How do you leak??
  ```C
  puts(*puts);
  ```
  * Execute puts with puts got offset and it will print the libc address of puts
* _What_ version of libc?
  * Leak the address: and then calculate offset to desired libc function (or gadget)
  * [Libc DB](http://libcdb.com)

**Extensions**
* ROP chains can become very large, and you may not be able to fit them all in your buffer overflow
* Can move stack somewhere else that is `-wx`
* Use partial/complete overwrite of esp to move stack
* Use a `ret` sled and jump back to your own buffer (make sure you return to a 4-byte aligned address)
* _one gadget_ is a series of instructions that result in a shell (usually requires some registers to be set up correctly beforehand)
  * useful if you have limited space or only space to execute one gadget
  * `one_gadget` program will find them in libc
