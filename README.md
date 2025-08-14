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

However, there are huge bottlenecks in these designs, **they do work and work well** for what they are but I wanted something:
* faster,
* more sophisticated and
* more modular in it's design.

## The Issues with current designs

In basic point form, the issues that are common to most TTL CPU designs are:

* EPROMs have delay times of upwards of 100-150ns or worse.
* Single Bus designs cause a bottleneck, particularly with ALU operations.
* ALU operations often rely on [74181](https://www.righto.com/2017/03/inside-vintage-74181-alu-chip-how-it.html) or the [74283](https://www.ti.com/lit/gpn/SN54S283) 4-bit binary adder with support chips.
* Single Instruction fetch/decode and execute phase.
* Single control unit with complex control line distribution.
* Registers as basic latches.

There are some slightly more advanced designs, that have one or more of these features:

* An ALU bus directly connected to some of the system registers.
* Registers implemented using counter chips allowing Increment / Decrement operations.
* ALU implemented as logic gates with mux chips removing the reliance on vintage End of life chips like the 74181
* Dual stage Pipeline with simple logic decoding.
* MMU like features to address memory beyond Address Buss range.

# CPU Features

In order to make the CPU faster the design will need to eliminate the slowest components, particulary EPROMS.
The following are where performance can be improved allowing multiple tasks to be performed within the same clock cycles:

## Implement Microcode in SRAM 

We need to Backfill the SRAM's on reset from an EPROM. Result, Microcode code access goes from 120ns to 10ns depending on chip selection. Implement on reset, using a special sequencer to perform SP->RD->WR->INC_SP-> repeat sequence, then when complete, switch the buffers for the EPROM to open collector and the SRAM's take effect.
  
## R-BUS
Rather than a single buss for register access, provide two or more busses, This will give all register access to a separate R-Bus for register to register moves. We can dedicate a register as the ALU results register maybe R7

## Instruction Groups

Using an idea from the MIPS CPU, use 2 bits as a "I-Type" field and have four separate Instruction registers (using a 74x139), control logic can then be implemented on groups of related instructions. This allows both a parallel pipleline and simplifies the control logic to handle just the signals for the instructions to be handled by that pipeline.

## Advanced Registers

Influenced by the MIPS CPU again, my initial design has registers R0 to R7. Having 8 registers means having 3 bits for a source and 3 bits for a destination register format. Rather than have R0 always return zero, R0-R7 will include the ability to perform the following instructions that would typically be handled by an ALU:
* INC / DEC
* BSET / BCLR / BTST
* Shifts / Rotates
* Zero / Invert / Bitwise OR / AND / XOR operations
  
The registers would have a secondary register latch to enable ALU oeprations to be performed, the 2nd latch is loaded by an XFER instruction XFER Rs, Rd This would be different to a LD instruction which moves data into the "A" Latch of a Register.


## Four wide Instuction Processing

## Separate Stack RAM

Why does stack ram need to be part of main memory? I asked myself this many times, logcally a Stack is built from the top down and RAM usage grows upwards. If they meet, you are in trouble, but generally they are separate conceptually but share the same chips. If you split the stack RAM into a 32K chip make the SP register output to a "local" SP bus via some counter chips read 74x161/74x191/193 etc, then include the control logic to manage PUSH, POP,CALL, RET and RSP (Reset SP) then you can do stack operations from registers while still having the Instruction Data Bus fetech and process more instructions.





# ISA

# Software

As I narrowed down on the width of the CPU, I decided an assembler project would highlight any issues as well as clarify the hardware needed to support the ISA. So I developed an Assembler. 
Having written 8080/8085/z80 and 68020 assembler code back in the 70's and 80's as an Embedded Systems Engineer in my younger days, I decided I would borrow ideas from these CPU's as well as the MIPS and RISC-V designs.

Soon to be released Assembler: https://github.com/z900collector/CPU32-Assembler

# Schematics

Still being finalized as I build modules. Will be released in KiCad and PDF formats soon.
