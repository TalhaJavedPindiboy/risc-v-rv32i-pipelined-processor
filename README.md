# 5-Stage Pipelined RISC-V (RV32I) Processor in Verilog

A synthesizable, modular 32-bit RISC-V processor core implementing a classic 5-stage instruction pipeline in Verilog HDL. The core features full hazard handling—including forwarding, load-use detection stalls, and control-hazard flushes—and is verified using custom testbenches in Xilinx Vivado.

---

## Architecture Overview

The core executes instructions across five discrete pipeline stages separated by pipeline registers:

1. **Instruction Fetch (IF):** Program counter management with sequential incrementing (`+4`), target updating for branch/jump instructions, and stall capability.
2. **Instruction Decode (ID):** Instruction decomposition, 32x32-bit register file reads (with hardwired `x0 = 0`), sign-extended immediate generation across standard formats (I, S, B, U, J), and main/ALU control decoding.
3. **Execute (EX):** 32-bit Arithmetic Logic Unit (ALU) operations, dedicated branch comparator, jump address calculation, and operand forwarding multiplexers.
4. **Memory Access (MEM):** Word-aligned synchronous data memory interface supporting store and asynchronous load access.
5. **Write Back (WB):** Multiplexed write-back path to select between ALU results, memory read data, link address (`PC+4`), or upper immediates.

### Hazard Management
* **Data Hazards (RAW):** Resolved via an Execute-stage **Forwarding Unit** that routes updated register data directly from `EX/MEM` and `MEM/WB` stages back to ALU inputs.
* **Load-Use Hazards:** Mitigated by a **Hazard Detection Unit** that detects read dependencies on memory load destinations in `ID/EX`, asserting a single-cycle stall on the PC/IF-ID registers and inserting a bubble into `ID/EX`.
* **Control Hazards:** Addressed by flushing invalid instructions in `IF/ID` and `ID/EX` upon confirmed taken branches (`BEQ`, `BNE`, `BLT`, `BGE`, `BLTU`, `BGEU`) or jumps (`JAL`, `JALR`).

---

## Supported Instruction Set

* **R-Type:** `ADD`, `SUB`, `SLL`, `SLT`, `SLTU`, `XOR`, `SRL`, `SRA`, `OR`, `AND`
* **I-Type:** `ADDI`, `SLTI`, `SLTIU`, `XORI`, `ORI`, `ANDI`, `SLLI`, `SRLI`, `SRAI`, `LW`, `JALR`
* **S-Type:** `SW`
* **B-Type:** `BEQ`, `BNE`, `BLT`, `BGE`, `BLTU`, `BGEU`
* **U-Type:** `LUI`, `AUIPC`
* **J-Type:** `JAL`

---

## File Structure

```text
├── rtl/
│   ├── alu.v                 # 32-bit Arithmetic Logic Unit
│   ├── alu_decoder.v         # ALU operation decoder based on funct3/funct7/opcode
│   ├── branch_comp.v         # Dedicated branch condition comparator
│   ├── controller.v          # Main opcode decoder and datapath control generator
│   ├── data_mem.v            # Synchronous RAM data memory
│   ├── imm_gen.v             # Immediate generator and sign-extender
│   ├── prg_mem.v             # Instruction ROM memory containing test assembly
│   ├── reg_file.v            # 32x32-bit general-purpose dual-read register file
│   ├── fetch_stage.v         # Program counter logic and fetch multiplexers
│   ├── forwarding_unit.v     # EX/MEM and MEM/WB operand forwarding logic
│   ├── hazard_unit.v         # Load-use hazard detection unit
│   ├── pipeline_regs/        # Pipeline registers: IF_ID, ID_EX, EX_MEM, MEM_WB
│   └── riscv_top.v           # Top-level datapath and control interconnect
└── sim/
    └── tb_riscv_top.v        # Behavioral simulation testbench with signal probes
