---
layout: post
categories: Week04
---

## Format Strings

> The format string specifies how subsequent arguments are converted to output

~ Man Page

`%n`: Writes number of characters written so far to the argument (i.e. not to the output stream)

* To avoid null byte problem, put the data at the end, and ensure all format arguments are at the beginning (so that they get executed before the printing terminates)

## Where is libc?

**PLT**: Process Linkage Table
* Call external functions whose addresses are loaded in dynamically

**GOT**: Global Offset Table
* 'Refernce Table' for where various libc functions can be found
* Initially, entirely populated with pointers back to `PLT`.
	* Jump to `ld.so` (Dynamic linker/loader), which then finds and loads the required function
* We can manipulate the GOT to point to our own functions instead of library functions	
	* GOT should not be writable, but it is (because things aren't linked at launch, they are dynamically linked at runtime).
