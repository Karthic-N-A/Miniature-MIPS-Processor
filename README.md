# Miniature MIPS Processor

## Overview

A 32-bit MIPS processor built in Verilog. Designed with an optimized multi-cycle datapath, this processor was successfully synthesized, implemented, and deployed on a PYNQ-Z2 FPGA as part of the CS220 (Computer Organization) course at IIT Kanpur under the supervision of Prof. Mainak Chaudhuri. 

---

## Instruction Encoding

Each MIPS instruction is 32 bits long. The processor decodes these standard formats combinationally to extract the appropriate fields.

* **R-Format (Register):** Used for instructions with only register operands (add, sub, and, or, xor, nor, sll, srl, sra, sllv, srlv, srav, syscall, slt, sltu, jr, jalr).
* **Encoding:** `opcode (6) | rs (5) | rt (5) | rd (5) | sh_amt (5) | function (6)`
* **I-Format (Immediate):** Used for instructions with an immediate operand (addi, andi, ori, xori, lw, sw, lb, sb, lh, sh, lbu, lhu, lui, beq, bne, bltz, bgez, blez, bgtz, slti, sltiu).
* **Encoding:** `opcode (6) | rs (5) | rt (5) | immediate (16)`
* **J-Format (Jump):** Used for unconditional jumps (j, jal).
* **Encoding:** `opcode (6) | jump_target (26)`

## Three-Cycle Execution Flow

To avoid critical path timing failures, the processor operates as a FSM requiring three cycles to complete an instruction. A new instruction is fetched only after the previous instruction is completed.

* **State 0 - Fetch, Decode, Read RF:** The processor fetches an instruction on the positive edge of the clock. It decodes the instruction to extract the fields (`opcode`, `func`, `shift_amount`) and reads the register file to fetch operands (`src1`, `src2`).
* **State 1 - Execute:** The ALU performs the required arithmetic, logical operation, or branch target computation. System calls (`syscall`) and memory address calculations are also executed in this cycle.
* **State 2 - Writeback and PC update:** The computed result from the ALU, or the data loaded from memory, is written back to the register file. Branch conditions update the Program Counter (PC) accordingly.

## Big-Endian Aligned Load/Store Architecture

To support byte and half-word operations, custom extraction and alignment logic was built into the datapath:

* **Store Operations (sb, sh):** For sub-word stores, the processor reads the existing 32-bit word from memory; dynamically modifies the appropriate bits based on the address offset, splices in the new sub-word from the register file, and issues a write-back command.
* **Load Operations (lb, lbu, lh, lhu):** The processor extracts the correct byte or half-word from the fetched 32-bit `load_value`. It automatically applies sign-extension for signed loads (lb, lh) or zero-extension for unsigned loads (lbu, lhu) before writing it to the destination register.

## Key Architectural Features

* **Hardware I/O Stalling:** A circular I/O register array tracks outputs. The processor generates an `io_stall` signal upon the fifth consecutive print syscall, safely freezing execution until the environment acknowledges copying the registers via a `copied_io_regs` flag.
* **Keyboard Input Handshake:** A custom syscall (`SYS_read` 1003) pauses the FSM and asserts `waiting_for_input`. The environment responds by supplying a 32-bit value alongside an `input_value_valid` flag, smoothly resuming execution.
* **Combinational ALU:** The Arithmetic Logic Unit is entirely combinational - executing shifts, arithmetic, and branch evaluations in a single block. 
* Hardwired `$0` register.
