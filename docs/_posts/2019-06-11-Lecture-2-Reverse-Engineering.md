---
layout: post
categories: Week02
---
## Reverse Engneering: Why?
> Reversing: Pulling apart existing software, figuring out how it works, and then breaking it

* Security Testing: to verify something fits the specification, or find vulns.
* Malware: Work out how it works and stop it
  * Can patch and remove jumps etc.
* Some _crypto_ can be easily bypassed

### Compilation
* Source code > Compiler > ASM > Assembler > Machine Code > Linker > Program
* Program > Loader > Process

## x86 Architecture
**Syntax:**
Use Intel (~~AT&T~~)
```bash
echo "set disassembly-flavor intel" >> ~/.gdbinit
```

### Registers
![]({{ site.baseurl }}/assets/images/x86-registers.png)

'E' stands for 'extended'.

**Register Conventions**
* `EBX`: "Base index" for arrays
* `ECX`: "Counter"
* `EIP`: Instruction Pointer
* `EAX`, `EBX`, `ECX`, `EDX`, `ESI`, `EDI`: General Purpose
* `EAX`: is frequently used as the first argument of a funtion call, and as return value
* `ESP`: "Stack Pointer"
* `EBP`: "Base Pointer"
* `ESI`: "Source"
* `EDI`: "Destination

### A Basic Function Call
![]({{ site.baseurl }}/assets/images/stack.png)

```c
void drawSquare (args) {

  // stuff
  drawLine(more, args);
  // more stuff

}

drawSquare();
```

**Assembly Process for Calling Functions**
```asm
PUSH EBP,
MOV EBP, ESP

# reserve space for arguments
ADD ESP, 8    # e.g. 8 byes for 2x4byte numbers
```

### Instructions

| Instruction | Description |
| --- | --- |
| `NOP`| 0x90: No Operation |
| `PUSH eax` |  pushes value in `eax` onto top of stack |
| `POP eax` | takes value on top of stack and pops it into `eax`|
| `MOV` | Moves data. `[]` mean dereference address in register|
| `ADD` | Adds second argument to the first argument|
| `SUB` | Subtract|
| `LEA` | Load Effective Address: load the address of the source into destination|
| `JMP` | Essentially set EIP to a certain address. Jumps are relative unless you specify a register|
| `CALL` | Can be relative or non-relative. Essentially performs `PUSH eip` then `JMP addr`|
| `RET` | Is the opposite of CALL: it is essentially "`POP eip`". Can take an argument (which means cleans up n bytes on the stack, then pops eip - rarely use).|
| `INT`, `SYSENTER`, or `SYSCALL` | Perform syscall|
| `REP` | Repeat instruction. Takes argument to which instruction you want to repeat|
| `AND`, `OR` etc. | Operations |
| `TEST` | same as an AND: doesn't save values, sets 0 flag |
| `CMP` | same as SUB: doesn't save values, sets 0 flag |
| Conditional Jumps | Based on some flag: e.g. jump if zero, jump if equal |

### Calling Conventions
#### fastcall
* Uses registers to pass arguments
* Common in windows

#### stdcall
* Pushes the arguments onto the stack
* **Callee** is responsible for cleaning up the stack

#### cdecl
* Pushes argument onto the stack
* **Caller** is responsible for cleaning up the stack
* This is what linux uses
* Return value is normally in `eax`

## Stack Protections
`checksec` is a pwntools command to check what protections are in place on a binary.

**Stack Canary/Cookie**
* Random number before protected area on stack (such as return address)
  * If the random number has changed, the program will crash instead of return
* _Bypass_
  * Find out the canary value and just write it in the exploit so it still apperas the same
* Format of Stack Canaries:
  * On a 32-bit machine, it's a 4byte value with a null termination: this makes it hard to exploit via string copy functions since strings will all end in a null
  * Randomised on Program Start: but every function will have the same canary during execution
  * `GS:0x14` is accessing a struct containing `uintptr_t stack_guard;`, which is the canary


**Variable Reordering**
* During compilation, varibales are reordered to make buffers at the bottom of the stack, with other variables on top (so other vars can't be overwritten)

**ASLR**
* Still to come...

`__x86.get_pc_thunk.bx`:
  *  Get program counter 
  * Way of findin th ebase of the binary
  * "Thunk": subroutine used to inject additional calculation into another subroutine. 

**NX: Non-executable Stack**
* Should never have a region of memory that is writeable AND executable
* Can be disabled and isn't used in some places
  * e.g. all browsers need to be able to write and then execute: case in point: javascript

## Reverse Engineering Techniques
### Structures & Arrays
**Array**
```asm
mov		eax, [esi+edi*4+18h] ; accessing an entry of the array esi at index edi. The 18h is because this example is an array inside of a struct (0x18 is start of array in struct)
```
```C
struct mystruct  {
	int age;
	char * name;
	struct mystruct* next;

	int uid;
	unt gid;
	int array[100];
}
```

So the above assembly is the same as this:
```C
struct.array[edi];
```

**Recognising Data Structures**
If a function is already reversed, can use this knowledge to work out data structuers of caller functions, since they will have to pass in arguments of the correct type.

**Switch Statements**
* Forms an array of addresses 'jump table'
  * If the cases do not start at 0, then there is a subtraction so that the same jump table/offsets can be used
* If it is not an integer, then it will use a less/grrater than tree

#### Recommended Reading
* Smashing the STack for Fun and Profit (_Phrack_)
