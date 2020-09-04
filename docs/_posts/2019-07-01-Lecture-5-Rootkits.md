---
layout: post
categories: Week05
---

## What is a Rootkit?

* Achieve root access on a system
* May want to persist
* Aim to hide presence to the system user

## Userland Kits

* Changing widely used programs (e.g. ls)
* often to hide other aspects
* infect some process and run from them

## Kernel Kits

> **Hooking** Redirect code execution to your own code

### Type 1

**Syscall Table**
* Array of function pointers (syscall id is an index into this table)
* Can easily replace syscalls to perform extra checks and run your own code on calls

**Interrupt Descriptor Table**
* Used for handling different types of interrupts
* Div0 handler, pagefault handler

Changing these are simple, and easy to detect

### Type 2
* Manipulate internal kernel data structures
* Hook _special_ files (such as `/proc` and `/dev`) - these call functions when read (and they are dynamic)

## Detection
* Userland: signature everything
* Kernel: check addresses are in sane locations
	* Check data structures vs what a process tells you about something (e.g. files in dir)
	* `tasklist`: in linux, every new process will have one, so the rootkit will have one of these
		* Look for things that the rootkit author can't destroy