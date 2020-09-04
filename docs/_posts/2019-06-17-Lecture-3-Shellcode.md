---
layout: post
categories: Week03
---
## Shellcode
Various general types:
* Connect back: get the remote target to connect back to you
* Socket reuse: reuse something that is/was already running on the target
* Egghunter: Searches for a signature in memory and then jumps to it (e.g. put shellcode in another region)
* Second Stage download: Download a larger payload to run (initial shellcode has restricted size

**Special Characters**
* Sometimes the shellcode can't have certain bytes such as \x00 or \r \n in HTTP protocol
* Need to work around these restrictions: 
	* e.g. to set value to zero, xor with itself instead
	* May be restricted to printable characters


### Syscalls
1. Put syscall number into `eax`
* Call `int 0x80` - interrupt with code 80

Syscall arguments are placed in registers in order (ebx, ecx...)

[Syscall Reference]('http://cgi.cse.unsw.edu.au/~z5164500/syscall/')

**Finding where you are**
* Your shellcode won't know where it is in memory
* Can find it with something like this:

```asm
call blah

blah:
	pop eax ; eax now contains eip
```

### Nop Sled
```
\x09\x09\x09\x09\x09\x09\x09\x09\x09\x09\x09\x09\x09\x09\x09\x09\x09\x09\x09\x09
```
* Some WAPs will block payloads containing long `\x90` strings, but there are ways of getting around it:
	* e.g. use `inc eax` or `push; pop` instead


### Different Shelcode Approaches
**Egghunter**
Omlettte: A variant with multiple small segments that are stitched together

**Non-ex stack**
* Fix memory permissions may need to map a new rwx page in memory

**Syscall Proxy**
* Run binary locally, and send syscalls for exe on remote
* Hard to detect 

**Mosdef**
* Python compiler in shellcode to execute python on remote target

**Userland Exec**
* Execute a binary without using the kernel:
	* Map and load binary into memory of the current program
* Hard to detect: since it isn't going to start a new process
