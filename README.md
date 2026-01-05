4-Bit Sequential Arithmetic ALU

FPGA-Based Digital Logic Design on DE1-SoC

A complete Arithmetic Logic Unit (ALU) designed and implemented in VHDL, featuring 11 operations including combinational logic, arithmetic, and sequential functions with real-time 7-segment display output.

----------------------------------------------------------------------------------

Overview

This project implements a fully functional 4-bit ALU synthesized on an Intel/Altera DE1-SoC FPGA board. The system processes two 4-bit inputs through user-selectable operations and displays results on a 7-segment display in real-time. The design combines both combinational logic (gates, arithmetic, comparison) and sequential logic (shift registers, bidirectional counter).

- Platform: DE1-SoC (Cyclone V FPGA)
- Language: VHDL
- Tools: Intel Quartus Prime

<img width="687" height="731" alt="image" src="https://github.com/user-attachments/assets/117b54d3-1d89-4e9b-8ed5-85b5e121ab81" />

-----------------------------------------------------------------------------------


Supported Operation

| OpCode | Operation | Type | Description |
|:------:|-----------|------|-------------|
| `0000` | AND | Combinational | Bitwise AND of A and B |
| `0001` | OR | Combinational | Bitwise OR of A and B |
| `0010` | XOR | Combinational | Bitwise XOR of A and B |
| `0011` | INV (NOT) | Combinational | Bitwise inversion of A |
| `0100` | ADD | Combinational | Arithmetic addition (A + B) with overflow |
| `0101` | COMPARE | Combinational | Outputs: Greater (001), Equal (010), Less (100) |
| `0110` | IsOdd | Combinational | Returns 1 if A is odd |
| `0111` | IsEven | Combinational | Returns 1 if A is even |
| `1000` | Shift Left | Sequential | Logical left shift (×2) |
| `1001` | Shift Right | Sequential | Logical right shift (÷2) |
| `1010` | Counter | Sequential | Bidirectional counter with load |


Hardware Interface

Input Mapping(DE1-SoC)

| Signal | Width | Mapped To | Description |
|--------|-------|-----------|-------------|
| A | 4-bit | SW[3:0] | Primary operand |
| B | 4-bit | SW[7:4] | Secondary operand |
| OpCode | 4-bit | SW[17:14] | Operation selector |
| CLK | 1-bit | KEY[0] | Clock for sequential ops |
| RST | 1-bit | KEY[1] | Global reset |
| Enable | 1-bit | SW[8] | Enable sequential operations |
| Direction | 1-bit | SW[9] | Counter direction (0=Up, 1=Down) |


Output Mapping

| Signal | Width | Mapped To | Description |
|--------|-------|-----------|-------------|
| Result | 4-bit | LEDR[3:0] | Binary result |
| 7-Seg | 7-bit | HEX0[6:0] | Hexadecimal display |
| Overflow | 1-bit | LEDR[9] | Adder overflow indicator |

------------------------------------------------------------------------------------

Module Description

Combinational Modules

Logic Gates (AND, OR, XOR, INV)

Direct bitwise operations on 4-bit vectors. Each module uses VHDL's native vector operations for clean, synthesizable code.


4-Bit Adder with Overflow Detection

Uses 5-bit internal signal to capture carry-out. Inputs are zero-extended before addition; bit 4 of the result provides the overflow flag.


Comparator

Casts inputs to unsigned type for proper magnitude comparison. Outputs three mutually exclusive flags: GT (A>B), EQ (A=B), LT (A<B), encoded as a 3-bit result.


Parity Checker (Odd/Even)

Lightweight single-bit check on LSB: A(0) = 1 → odd, A(0) = 0 → even.


Sequential Modules

Shift Registers

On rising clock edge:

- Shift Left: Q <= Q(2 downto 0) & '0' (multiply by 2)
- Shift Right: Q <= '0' & Q(3 downto 1) (divide by 2)


Bidirectional Counter

Features synchronous reset, parallel load, and enable control:

- RST = 1: Counter clears to 0000
- LOAD = 1: Counter loads value from input A
- EN = 1, DIR = 0: Count up
- EN = 1, DIR = 1: Count down

Display Driver

7-Segment Decoder

Combinational case statement mapping 4-bit binary to active-low segment patterns for hexadecimal display (0-F).

----------------------------------------------------------------

Project Structure

```
fpga-4bit-alu/
├── VHDL code
│   ├── and_gate.vhd          # Bitwise AND
│   ├── or_gate.vhd           # Bitwise OR
│   ├── xor_gate.vhd          # Bitwise XOR
│   ├── inv_gate.vhd          # Bitwise NOT
│   ├── adder.vhd             # Adder with overflow
│   ├── comparator.vhd        # Magnitude comparator
│   ├── odd_even.vhd          # Odd/Even detection
│   ├── shift_left.vhd        # Left shift register
│   ├── shift_right.vhd       # Right shift register
│   ├── counter.vhd           # Up/Down counter
│   ├── seven_seg.vhd         # Display decoder
│   └── ALU_LSD.vhd           # Top-level integration  
├── docs/
│   ├── block_diagram.png
│   ├── schematic.png
│   └── waveforms/            # ModelSim output
└── README.md
```

------------------------------------------------------------------------------------

Verification

Simulation

All operations verified using ModelSim waveform simulation before hardware deployment.


Hardware Test Results

| Test | A | B | OpCode | Expected | Actual | Status |
|------|------|------|--------|----------|--------|:------:|
| AND | 0101 | 0011 | 0000 | 0001 (1) | 0001 (1) | ✅ |
| OR | 0101 | 0011 | 0001 | 0111 (7) | 0111 (7) | ✅ |
| XOR | 0101 | 0011 | 0010 | 0110 (6) | 0110 (6) | ✅ |
| INV | 0101 | XXXX | 0011 | 1010 (A) | 1010 (A) | ✅ |
| ADD | 0011 | 0101 | 0100 | 1000 (8) | 1000 (8) | ✅ |
| CMP (A>B) | 1010 | 0101 | 0101 | 0001 | 0001 | ✅ |
| IsOdd | 0101 | XXXX | 0110 | 0001 | 0001 | ✅ |
| IsEven | 0100 | XXXX | 0111 | 0001 | 0001 | ✅ |
| SHL | 0011 | XXXX | 1000 | 0110 (6) | 0110 (6) | ✅ |
| SHR | 1000 | XXXX | 1001 | 0100 (4) | 0100 (4) | ✅ |
| Counter | 0010 | XXXX | 1010 | 0011 (3) | 0011 (3) | ✅ |

--------------------------------------------------------------------------------------

Demo

https://www.youtube.com/watch?v=-aLPYIEexU0

The video demonstrates real-time operation of all test cases on the DE1-SoC board, showing switch inputs, LED outputs, and 7-segment display results.
--------------------------------------------------------------------------------------

Build Instructions

Requirements

- Intel Quartus Prime (Lite or Standard)
- DE1-SoC Board (or compatible Cyclone V device)
- USB-Blaster for programming

Compilation

1. Open Quartus Prime and create new project
2. Add all .vhd files from src/ directory
3. Set alu_top.vhd as top-level entity
4. Import pin assignments from de1soc_pins.qsf
5. Compile (Processing → Start Compilation)
6. Program device (Tools → Programmer)

--------------------------------------------------------------------------------------

Key Learnings:

- Modular VHDL Design: Breaking complex systems into reusable components
- Combinational vs Sequential Logic: Understanding when state is required
- FPGA Synthesis: Translating HDL to actual hardware resources
- Hardware Debugging: Using LEDs and displays for real-time verification
- Timing Considerations: Clock edge sensitivity in sequential circuits

--------------------------------------------------------------------------------------

Refernces:

- DE1-SoC User Manual — Intel/Altera
- EE-211 Logic System Design Laboratory Manual
- Nandland: Shift Operations
- AMD Vivado Synthesis Guide

--------------------------------------------------------------------------------------

Lisense 

This project was completed as part of academic coursework. Feel free to reference for educational purposes.



