# RX3588 Processor & VM
A virtual machine based on an imaginary RISC processor, the RX3588.

## Overview
The RX3588 is a virtual processor, implemented in Java, along with the other
components (RAM, I/O controller, interrupt controller, etc.) necessary to make
a working system.

### Goals of the RX3588:
* RISC; very few instructions (31)
* Small number of registers (8)
* 8-bit opcodes with embedded operands
* 32-bit addressing and registers (max 2GB memory)
* 64K I/O ports
* Multi-mode (programs run in either Administrative or User mode)
* Base-addressing (allowing relocatable modules)
* Queued interrupt controller

You'll notice many of these goals are about providing *minimal* functionality.
This is because the processor and system are intended to be used as a learning
opportunity: You will, for example, need to implement your own software
multiplication routine, as there's no MUL instruction. You will need to be
efficient in your use of registers and use the stack effectively. You will need
to manage addressing and modes properly, so your OS runs in Administrative mode
and your application software in User mode.

## Technical Details
The RX3588 has 8 registers and only 31 instructions. All necessary functionality
for an OS and application software can be built on these.

### Registers
|Register|Type|Used For|User Mode|
|---|---|---|---|
|X|General Purpose|Load/store values, arithmetic, logic, jumps, calls|R/W|
|Y|..|..|R/W|
|M|Memory Address|Load/store addresses|R/W|
|S|Stack Address|Points to top of stack|R/W|
|F|Flags|Indicates CPU status, control CPU|R/W (masked)|
|B|Base|Absolute address of start of process's memory area|R|
|E|Extent|Absolute address of end of process's memory area|R|
|A|Administrative Stack Address|Absolute address of Administrative mode stack|-|

### Instruction Set

|Instruction|Bits|Operands|Description|Administrative?|
|---|---|---|---|---|
|LD r, [M]|00000rrr|**r:** X, Y, M, S, F, B, E, A|Load value at memory location in M into register r||
|ST [M], r|00001rrr|**r:** X, Y, M, S, F, B, E, A|Store value in register r into memory location in M||
|PSH r|00010rrr|**r:** X, Y, M, S, F, B, E, A|Push value in register r onto top of stack||
|POP r|00011rrr|**r:** X, Y, M, S, F, B, E, A|Pop value off top of stack into register r||
|ADD d, s|00100dss|**d:** X, Y<br/>**s:** X, Y, M, S|Add value in register s to value in register d, storing result in register d||
|ADC d, s|00101dss|**d:** X, Y<br/>**s:** X, Y, M, S<br/>*s and d cannot be same register*|Add value in register s and value of carry flag to value in register d, storing result in register d||
|SUB d, s|00110dss|**d:** X, Y<br/>**s:** X, Y, M, S<br/>*s and d cannot be same register*|Subtract value in register s from value in register d, storing result in register d||
|SBB d, s|00111dss|**d:** X, Y<br/>**s:** X, Y, M, S<br/>*s and d cannot be same register*|Subtract value in register s and value of borrow (carry) flag from value in register d, storing result in register d||
|INC r|010000rr|**r:** X, Y, M, S|Increment (add one to) value in register r, storing result in register r||
|DEC r|010001rr|**r:** X, Y, M, S|Decrement (subtract one from) value in register r, storing result in register r||
|PUT r, i|01001rrr|**r:** X, Y, M, S, F, B, E, A|Put immediate (literal) value i into register r||
|AND d, s|01010dss|**d:** X, Y<br/>**s:** X, Y, M, S<br/>*s and d cannot be same register*|Perform bitwise logical and between registers d and s, storing result in register d||
|OR d, s|01011dss|**d:** X, Y<br/>**s:** X, Y, M, S<br/>*s and d cannot be same register*|Perform bitwise logical or between registers d and s, storing result in register d||
|XOR d, s|01100dss|**d:** X, Y<br/>**s:** X, Y, M, S|Perform bitwise logical exclusive-or between registers d and s, storing result in register d||
|NOT r|011010rr|**r:** X, Y, M, S|Perform bitwise negation (ones complement) of value in register r, storing result in register r||
|JMP r|011100rr|**r:** X, Y, M, S|Jump to memory address in register r||
|JC r|011101rr|**r:** X, Y, M, S|Conditionally jump to memory address in register r, if carry flag set||
|JS r|011110rr|**r:** X, Y, M, S|Conditionally jump to memory address in register r, if sign flag set||
|JZ r|011111rr|**r:** X, Y, M, S|Conditionally jump to memory address in register r, if zero flag set||
|CALL r|100001rr|**r:** X, Y, M, S|Push current instruction pointer address onto top of stack and jump to memory address in register r||
|RET|10001000|-|Pop memory address off top of stack and jump to that address||
|HLT|10001001|-|Permanently disable interrupts (until power cycled) and halt execution of CPU|X|
|SVC r|1000101r|**r:** X, Y|Push current instruction pointer address onto top of stack and place value in register r onto interrupt queue||
|LS r|100011rr|**r:** X, Y, M, S|Shift value in register r one bit position to left; set rightmost bit to 0; store result in register r||
|RS r|100100rr|**r:** X, Y, M, S|Shift value in register r one bit position to right; set leftmost bit to 0; store result in register r||
|RSS r|100101rr|**r:** X, Y, M, S|Shift value in register r one bit position to right; set leftmost bit to value of sign flag; store result in register r||
|LSC r|100110rr|**r:** X, Y, M, S|Shift value in register r one bit position to left; set rightmost bit to value of carry flag; store result in register r||
|RSC r|100111rr|**r:** X, Y, M, S|Shift value in register r one bit position to right; set leftmost bit to value of carry flag; store result in register r||
|IN d, [s]|10110dss|**d:** X, Y<br/>**s:** X, Y, M, S|Read 32-bit value into register d from I/O port number stored in lower 16 bits of register s|X|
|OUT s, [d]|10111dss|**d:** X, Y, M, S<br/>**s:** X, Y|Write 32-bit value from register s to I/O port number stored in lower 16 bits of register d|X|
|CP d, s|11dddsss|**d:** X, Y, M, S, F, B, E, A<br/>**s:** X, Y, M, S, F, B, E, A<br/>*s and d cannot be same register*|Copy value in register s into register d||

## License
Apache 2.0 (see [LICENSE.txt](LICENSE.txt) and [NOTICE.txt](NOTICE.txt) for details).
