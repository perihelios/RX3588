# RX3588 Processor & VM
A virtual machine based on an imaginary RISC processor, the RX3588.

## Overview
The RX3588 is a virtual processor, implemented in Java, along with the other
components (RAM, I/O controller, interrupt controller, etc.) necessary to make
a working system.

###Goals of the RX3588:
* RISC; very few instructions (31)
* Small number of registers (8)
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
to manage mode effectively so your OS runs in Administrative mode and your
application software in User mode.

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

## License
Apache 2.0 (see [LICENSE.txt](LICENSE.txt) and [NOTICE.txt](NOTICE.txt) for details).
