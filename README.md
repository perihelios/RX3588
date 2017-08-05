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
|LD r, [M]|00000rrr|**r:** X, Y, M, S, F, B, E, A|Load value at memory location M into register **r**||
|ST [M], r|00001rrr|**r:** X, Y, M, S, F, B, E, A|Store value in register **r** into memory location M||
|PSH r|00010rrr|**r:** X, Y, M, S, F, B, E, A|Push value in register **r** onto top of stack||
|POP r|00011rrr|**r:** X, Y, M, S, F, B, E, A|Pop value off top of stack into register **r**||
|ADD d, s|00100dss|**d:** X, Y<br/>**s:** X, Y, M, S|Add value in register **s** to value in register **d**, storing result in register **d**||
|ADC d, s|00101dss|**d:** X, Y<br/>**s:** X, Y, M, S<br/>*s and d cannot be same register*|Add value in register **s** and value of carry flag to value in register **d**, storing result in register **d**||
|SUB d, s|00110dss|**d:** X, Y<br/>**s:** X, Y, M, S<br/>*s and d cannot be same register*|Subtract value in register **s** from value in register **d**, storing result in register **d**||
|SBB d, s|00111dss|**d:** X, Y<br/>**s:** X, Y, M, S<br/>*s and d cannot be same register*|Subtract value in register **s** and value of borrow (carry) flag from value in register **d**, storing result in register **d**||
|INC r|010000rr|**r:** X, Y, M, S|Increment (add one to) value in register **r**, storing result in register **r**||
|DEC r|010001rr|**r:** X, Y, M, S|Decrement (subtract one from) value in register **r**, storing result in register **r**||
|PUT r, i|01001rrr|**r:** X, Y, M, S, F, B, E, A|Put immediate (literal) value **i** into register **r**||
|AND d, s|01010dss|**d:** X, Y<br/>**s:** X, Y, M, S<br/>*s and d cannot be same register*|Perform bitwise logical *and* between registers **d** and **s**, storing result in register **d**||
|OR d, s|01011dss|**d:** X, Y<br/>**s:** X, Y, M, S<br/>*s and d cannot be same register*|Perform bitwise logical *or* between registers **d** and **s**, storing result in register **d**||
|XOR d, s|01100dss|**d:** X, Y<br/>**s:** X, Y, M, S|Perform bitwise logical *exclusive-or* between registers **d** and **s**, storing result in register **d**||
|NOT r|011010rr|**r:** X, Y, M, S|Perform bitwise negation (ones complement) of value in register **r**, storing result in register **r**||
|JMP r|011100rr|**r:** X, Y, M, S|Jump to memory address in register **r**||
|JC r|011101rr|**r:** X, Y, M, S|Conditionally jump to memory address in register **r**, if carry flag set||
|JS r|011110rr|**r:** X, Y, M, S|Conditionally jump to memory address in register **r**, if sign flag set||
|JZ r|011111rr|**r:** X, Y, M, S|Conditionally jump to memory address in register **r**, if zero flag set||
|CALL r|100001rr|**r:** X, Y, M, S|Push current instruction pointer address onto top of stack and jump to memory address in register **r**||
|RET|10001000|-|Pop memory address off top of stack and jump to that address||
|HLT|10001001|-|Permanently disable interrupts (until power cycled) and halt execution of CPU|X|
|SVC r|1000101r|**r:** X, Y|Push current instruction pointer address onto top of stack and place value in register **r** onto interrupt queue||
|LS r|100011rr|**r:** X, Y, M, S|Shift value in register **r** one bit position to left; set rightmost bit to 0; store result in register **r**||
|RS r|100100rr|**r:** X, Y, M, S|Shift value in register **r** one bit position to right; set leftmost bit to 0; store result in register **r**||
|RSS r|100101rr|**r:** X, Y, M, S|Shift value in register **r** one bit position to right; set leftmost bit to value of sign flag; store result in register **r**||
|LSC r|100110rr|**r:** X, Y, M, S|Shift value in register **r** one bit position to left; set rightmost bit to value of carry flag; store result in register **r**||
|RSC r|100111rr|**r:** X, Y, M, S|Shift value in register **r** one bit position to right; set leftmost bit to value of carry flag; store result in register **r**||
|IN d, [s]|10110dss|**d:** X, Y<br/>**s:** X, Y, M, S|Read 32-bit value into register **d** from I/O port number stored in lower 16 bits of register **s**|X|
|OUT s, [d]|10111dss|**d:** X, Y, M, S<br/>**s:** X, Y|Write 32-bit value from register **s** to I/O port number stored in lower 16 bits of register **d**|X|
|CP d, s|11dddsss|**d:** X, Y, M, S, F, B, E, A<br/>**s:** X, Y, M, S, F, B, E, A<br/>*s and d cannot be same register*|Copy value in register **s** into register **d**||

### Opcodes

|Opcode & Operands|Binary|Decimal|Hex|Octal|
|---|---|---|---|---|
|LD X, [M]|00000000|0|00|000|
|LD Y, [M]|00000001|1|01|001|
|LD M, [M]|00000010|2|02|002|
|LD S, [M]|00000011|3|03|003|
|LD F, [M]|00000100|4|04|004|
|LD B, [M]|00000101|5|05|005|
|LD E, [M]|00000110|6|06|006|
|LD A, [M]|00000111|7|07|007|
|ST [M], X|00001000|8|08|010|
|ST [M], Y|00001001|9|09|011|
|ST [M], M|00001010|10|0A|012|
|ST [M], S|00001011|11|0B|013|
|ST [M], F|00001100|12|0C|014|
|ST [M], B|00001101|13|0D|015|
|ST [M], E|00001110|14|0E|016|
|ST [M], A|00001111|15|0F|017|
|PSH X|00010000|16|10|020|
|PSH Y|00010001|17|11|021|
|PSH M|00010010|18|12|022|
|PSH S|00010011|19|13|023|
|PSH F|00010100|20|14|024|
|PSH B|00010101|21|15|025|
|PSH E|00010110|22|16|026|
|PSH A|00010111|23|17|027|
|POP X|00011000|24|18|030|
|POP Y|00011001|25|19|031|
|POP M|00011010|26|1A|032|
|POP S|00011011|27|1B|033|
|POP F|00011100|28|1C|034|
|POP B|00011101|29|1D|035|
|POP E|00011110|30|1E|036|
|POP A|00011111|31|1F|037|
|ADD X, X|00100000|32|20|040|
|ADD X, Y|00100001|33|21|041|
|ADD X, M|00100010|34|22|042|
|ADD X, S|00100011|35|23|043|
|ADD Y, X|00100100|36|24|044|
|ADD Y, Y|00100101|37|25|045|
|ADD Y, M|00100110|38|26|046|
|ADD Y, S|00100111|39|27|047|
|-|00101000|40|28|050|
|ADC X, Y|00101001|41|29|051|
|ADC X, M|00101010|42|2A|052|
|ADC X, S|00101011|43|2B|053|
|ADC Y, X|00101100|44|2C|054|
|-|00101101|45|2D|055|
|ADC Y, M|00101110|46|2E|056|
|ADC Y, S|00101111|47|2F|057|
|-|00110000|48|30|060|
|SUB X, Y|00110001|49|31|061|
|SUB X, M|00110010|50|32|062|
|SUB X, S|00110011|51|33|063|
|SUB Y, X|00110100|52|34|064|
|-|00110101|53|35|065|
|SUB Y, M|00110110|54|36|066|
|SUB Y, S|00110111|55|37|067|
|-|00111000|56|38|070|
|SBB X, Y|00111001|57|39|071|
|SBB X, M|00111010|58|3A|072|
|SBB X, S|00111011|59|3B|073|
|SBB Y, X|00111100|60|3C|074|
|-|00111101|61|3D|075|
|SBB Y, M|00111110|62|3E|076|
|SBB Y, S|00111111|63|3F|077|
|INC X|01000000|64|40|100|
|INC Y|01000001|65|41|101|
|INC M|01000010|66|42|102|
|INC S|01000011|67|43|103|
|DEC X|01000100|68|44|104|
|DEC Y|01000101|69|45|105|
|DEC M|01000110|70|46|106|
|DEC S|01000111|71|47|107|
|PUT X|01001000|72|48|110|
|PUT Y|01001001|73|49|111|
|PUT M|01001010|74|4A|112|
|PUT S|01001011|75|4B|113|
|PUT F|01001100|76|4C|114|
|PUT B|01001101|77|4D|115|
|PUT E|01001110|78|4E|116|
|PUT A|01001111|79|4F|117|
|-|01010000|80|50|120|
|AND X, Y|01010001|81|51|121|
|AND X, M|01010010|82|52|122|
|AND X, S|01010011|83|53|123|
|AND Y, X|01010100|84|54|124|
|-|01010101|85|55|125|
|AND Y, M|01010110|86|56|126|
|AND Y, S|01010111|87|57|127|
|-|01011000|88|58|130|
|OR X, Y|01011001|89|59|131|
|OR X, M|01011010|90|5A|132|
|OR X, S|01011011|91|5B|133|
|OR Y, X|01011100|92|5C|134|
|-|01011101|93|5D|135|
|OR Y, M|01011110|94|5E|136|
|OR Y, S|01011111|95|5F|137|
|XOR X, X|01100000|96|60|140|
|XOR X, Y|01100001|97|61|141|
|XOR X, M|01100010|98|62|142|
|XOR X, S|01100011|99|63|143|
|XOR Y, X|01100100|100|64|144|
|XOR Y, Y|01100101|101|65|145|
|XOR Y, M|01100110|102|66|146|
|XOR Y, S|01100111|103|67|147|
|NOT X|01101000|104|68|150|
|NOT Y|01101001|105|69|151|
|NOT M|01101010|106|6A|152|
|NOT S|01101011|107|6B|153|
|-|01101100|108|6C|154|
|-|01101101|109|6D|155|
|-|01101110|110|6E|156|
|-|01101111|111|6F|157|
|JMP X|01110000|112|70|160|
|JMP Y|01110001|113|71|161|
|JMP M|01110010|114|72|162|
|JMP S|01110011|115|73|163|
|JC X|01110100|116|74|164|
|JC Y|01110101|117|75|165|
|JC M|01110110|118|76|166|
|JC S|01110111|119|77|167|
|JS X|01111000|120|78|170|
|JS Y|01111001|121|79|171|
|JS M|01111010|122|7A|172|
|JS S|01111011|123|7B|173|
|JZ X|01111100|124|7C|174|
|JZ Y|01111101|125|7D|175|
|JZ M|01111110|126|7E|176|
|JZ S|01111111|127|7F|177|
|-|10000000|128|80|200|
|-|10000001|129|81|201|
|-|10000010|130|82|202|
|-|10000011|131|83|203|
|CALL X|10000100|132|84|204|
|CALL Y|10000101|133|85|205|
|CALL M|10000110|134|86|206|
|CALL S|10000111|135|87|207|
|RET|10001000|136|88|210|
|HLT|10001001|137|89|211|
|SVC X|10001010|138|8A|212|
|SVC Y|10001011|139|8B|213|
|LS X|10001100|140|8C|214|
|LS Y|10001101|141|8D|215|
|LS M|10001110|142|8E|216|
|LS S|10001111|143|8F|217|
|RS X|10010000|144|90|220|
|RS Y|10010001|145|91|221|
|RS M|10010010|146|92|222|
|RS S|10010011|147|93|223|
|RSS X|10010100|148|94|224|
|RSS Y|10010101|149|95|225|
|RSS M|10010110|150|96|226|
|RSS S|10010111|151|97|227|
|LSC X|10011000|152|98|230|
|LSC Y|10011001|153|99|231|
|LSC M|10011010|154|9A|232|
|LSC S|10011011|155|9B|233|
|RSC X|10011100|156|9C|234|
|RSC Y|10011101|157|9D|235|
|RSC M|10011110|158|9E|236|
|RSC S|10011111|159|9F|237|
|-|10100000|160|A0|240|
|-|10100001|161|A1|241|
|-|10100010|162|A2|242|
|-|10100011|163|A3|243|
|-|10100100|164|A4|244|
|-|10100101|165|A5|245|
|-|10100110|166|A6|246|
|-|10100111|167|A7|247|
|-|10101000|168|A8|250|
|-|10101001|169|A9|251|
|-|10101010|170|AA|252|
|-|10101011|171|AB|253|
|-|10101100|172|AC|254|
|-|10101101|173|AD|255|
|-|10101110|174|AE|256|
|-|10101111|175|AF|257|
|IN X, [X]|10110000|176|B0|260|
|IN X, [Y]|10110001|177|B1|261|
|IN X, [M]|10110010|178|B2|262|
|IN X, [S]|10110011|179|B3|263|
|IN Y, [X]|10110100|180|B4|264|
|IN Y, [Y]|10110101|181|B5|265|
|IN Y, [M]|10110110|182|B6|266|
|IN Y, [S]|10110111|183|B7|267|
|OUT [X], X|10111000|184|B8|270|
|OUT [Y], X|10111001|185|B9|271|
|OUT [M], X|10111010|186|BA|272|
|OUT [S], X|10111011|187|BB|273|
|OUT [X], Y|10111100|188|BC|274|
|OUT [Y], Y|10111101|189|BD|275|
|OUT [M], Y|10111110|190|BE|276|
|OUT [S], Y|10111111|191|BF|277|
|-|11000000|192|C0|300|
|CP X, Y|11000001|193|C1|301|
|CP X, M|11000010|194|C2|302|
|CP X, S|11000011|195|C3|303|
|CP X, F|11000100|196|C4|304|
|CP X, B|11000101|197|C5|305|
|CP X, E|11000110|198|C6|306|
|CP X, A|11000111|199|C7|307|
|CP Y, X|11001000|200|C8|310|
|-|11001001|201|C9|311|
|CP Y, M|11001010|202|CA|312|
|CP Y, S|11001011|203|CB|313|
|CP Y, F|11001100|204|CC|314|
|CP Y, B|11001101|205|CD|315|
|CP Y, E|11001110|206|CE|316|
|CP Y, A|11001111|207|CF|317|
|CP M, X|11010000|208|D0|320|
|CP M, Y|11010001|209|D1|321|
|-|11010010|210|D2|322|
|CP M, S|11010011|211|D3|323|
|CP M, F|11010100|212|D4|324|
|CP M, B|11010101|213|D5|325|
|CP M, E|11010110|214|D6|326|
|CP M, A|11010111|215|D7|327|
|CP S, X|11011000|216|D8|330|
|CP S, Y|11011001|217|D9|331|
|CP S, M|11011010|218|DA|332|
|-|11011011|219|DB|333|
|CP S, F|11011100|220|DC|334|
|CP S, B|11011101|221|DD|335|
|CP S, E|11011110|222|DE|336|
|CP S, A|11011111|223|DF|337|
|CP F, X|11100000|224|E0|340|
|CP F, Y|11100001|225|E1|341|
|CP F, M|11100010|226|E2|342|
|CP F, S|11100011|227|E3|343|
|-|11100100|228|E4|344|
|CP F, B|11100101|229|E5|345|
|CP F, E|11100110|230|E6|346|
|CP F, A|11100111|231|E7|347|
|CP B, X|11101000|232|E8|350|
|CP B, Y|11101001|233|E9|351|
|CP B, M|11101010|234|EA|352|
|CP B, S|11101011|235|EB|353|
|CP B, F|11101100|236|EC|354|
|-|11101101|237|ED|355|
|CP B, E|11101110|238|EE|356|
|CP B, A|11101111|239|EF|357|
|CP E, X|11110000|240|F0|360|
|CP E, Y|11110001|241|F1|361|
|CP E, M|11110010|242|F2|362|
|CP E, S|11110011|243|F3|363|
|CP E, F|11110100|244|F4|364|
|CP E, B|11110101|245|F5|365|
|-|11110110|246|F6|366|
|CP E, A|11110111|247|F7|367|
|CP A, X|11111000|248|F8|370|
|CP A, Y|11111001|249|F9|371|
|CP A, M|11111010|250|FA|372|
|CP A, S|11111011|251|FB|373|
|CP A, F|11111100|252|FC|374|
|CP A, B|11111101|253|FD|375|
|CP A, E|11111110|254|FE|376|
|-|11111111|255|FF|377|

## License
Apache 2.0 (see [LICENSE.txt](LICENSE.txt) and [NOTICE.txt](NOTICE.txt) for details).
