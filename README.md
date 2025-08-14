# S16-TTL-CPU
A superscaller TTL CPU using HC/HCT Logic - 16 Bit version

# Introduction

**TODO**

# The current state of most TTL CPU Projects

Having studied a huge range of 8/16 and the odd 32 bit TTL projects on offer, I decided to do something more advanced than the current designs. 

Most simple TTL CPU designs have the following:

* a single data bus
* a single control unit
* EPROM/EEPROM based microcode design
* Single Instruction Register (IR)
* basic latched registers
* Stack Pointer RAM shared with general RAM.
* Limited memory range
* Fixed Machine Cycle times (i.e T0 - T1 - T2 -T3 etc) for ALL instructions.
* Sligtly more advanced designs have a modified harvard architecture. With separate RAM and EPROM address spaces.
* Low clock frequency typicall < 4MHz

However, there are huge bottlenecks in these designs, **they do work and work well** for what they are but I wanted something:
* faster >25Mhz <50Mhz,
* more sophisticated design and
* more modular in it's design.

## The Issues with current designs

In basic point form, the issues that are common to most TTL CPU designs are:

* EPROMs have delay times of upwards of 100-150ns or worse thus limiting clock timing.
* Single Bus designs cause a bottleneck, particularly with ALU operations.
* ALU operations often rely on [74181](https://www.righto.com/2017/03/inside-vintage-74181-alu-chip-how-it.html) or the [74283](https://www.ti.com/lit/gpn/SN54S283) 4-bit binary adder with support chips.
* Most have a single Instruction fetch/decode and execute phase.
* Single control unit with complex control line distribution.
* Registers as basic latches without any additional logic capability.

There are some slightly more advanced designs, that have one or more of these features:

* An ALU bus directly connected to some of the system registers.
* Registers implemented using counter chips allowing Increment / Decrement operations.
* ALU implemented as logic gates with mux chips removing the reliance on vintage End of life chips like the 74181
* Dual stage Pipeline with simple logic decoding.
* Dual bus support - limited but still usable.
* MMU like features to address memory beyond Address Buss range.

# CPU Features

In order to make the CPU faster, the design will need to implement a degree of parallelism and eliminate the slowest components, particulary EPROMS. To avoid the path of Intel like chips with instruction bloat, it needs to be RISC like.
The following are where performance can be improved allowing multiple tasks to be performed within the same clock cycles and reducing bottlenecks in the overall design:

## R-BUS
Rather than a single bus for register access, provide two or more buses, This will give all register's access to a separate R-Bus for register to register moves. We can dedicate a register as the ALU results register maybe R7

## Instruction Groups

Using an idea from the MIPS CPU, use 2 bits as a "I-Type" field and have four separate Instruction registers (think [74HC139](https://www.ti.com/lit/gpn/SN74HCT139)) and 74HC574 Latches, control logic can then be implemented on groups of related instructions. This allows both a parallel pipleline design and segregates the control logic needed to handle just the signals for the instructions to be handled by that pipeline.

ISA details to follow once Assembler project is completed. But basically 2 bits for the type and the remaining 6 bits for the range of instructions in each group

## Advanced Registers

Influenced by the MIPS CPU again, my initial design has registers R0 to R7. Having 8 registers means having 3 bits for a source and 3 bits for a destination register format. Rather than have the main controller/sequencer control the registers, it would signal them of an operation and the Register Controller does the task. This allows the Instruction fetch cycle to re-occur directly after the Register Control has been set (it would do it's task(s) during the fetch, decode cycle), then be ready at the Execute cycle for the next register operation if there was one directly after the current one. With 6 bits for SRC and DST Registers and 2 bits for a type register the design was to 

### Localised ALU functions

Rather than have R0 always return zero, R0-R7 will include the ability to perform the following instructions that would typically be handled by an ALU:
* INC / DEC
* BSET / BCLR / BTST
* Shifts / Rotates
* Zero / Invert / Bitwise OR / AND / XOR operations

### Secondary register latch
The registers would have a secondary register latch to enable the basic ALU operations to be performed lcoally, the 2nd latch is loaded by an XFER instruction (XFER Rs, Rd) This would be different to a LD instruction which moves data into the "A" Latch of a Register. The XFER would load "A" and "B" at the same time. A logic operation would then be an XFER, followed by an LD, followed by the ALU operation. Results get written back to the "A" latch or if coded, transfered to another register using the R-BUS.

## Separate Stack RAM

Why does stack RAM need to be part of main memory? I asked myself this many times, logcally a Stack is built from the top down and RAM usage grows upwards. If they meet, you are in trouble, but generally they are separate conceptually but share the same chips. If you split the stack RAM into a separate exclusive address space and have the SP register output to a "local" SP bus via some counter chips read 74x161/74x191/193 etc, then include the control logic to manage PUSH, POP, CALL, RET and RSP (Reset SP) then you can do stack operations from registers while still having the Instruction Data Bus fetch and process more instructions.

On reset (or via the RSP Instruction) the stack is initiallized to 0xFFFF using pull up resistors on the "D" inputs to the counters, the MR and /LOAD signals then sets the address, the output of the counters drivers the address bus for the SRAM. Only a call or return requires the SRAM to present data back to the Main Data Bus.

## Advanced ALU

With a number of functions integrated into the registers, the ALU can focus on Addition/Subtraction/Division and Multiplication. 

# Program Counter

Where designs have used the ALU to implement ADD/SUB operations for the program counter, that functionality would be built into the PC module.

## Implement Microcode in SRAM 

We need to Backfill the SRAM's on reset from an EPROM. Result, Microcode code access goes from 120ns+ to 10ns depending on chip selection. Implement on reset, using a special sequencer to perform SP->RD->WR->INC_SP-> repeat sequence, then when complete, switch the buffers for the EPROM to open collector and the SRAM's take effect.



# ISA

# Software

As I narrowed down on the width of the CPU, I decided an assembler project would highlight any issues as well as clarify the hardware needed to support the ISA. So I developed an Assembler. 
Having written 8080/8085/z80 and 68020 assembler code back in the 70's and 80's as an Embedded Systems Engineer in my younger days, I decided I would borrow ideas from these CPU's as well as the MIPS and RISC-V designs.

Soon to be released Assembler: https://github.com/z900collector/CPU32-Assembler

# Schematics

Still being finalized as I build modules. Will be released in KiCad and PDF formats soon.
