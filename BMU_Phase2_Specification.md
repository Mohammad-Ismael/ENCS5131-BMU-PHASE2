> # ENCS5131, HARDWARE DESIGN LAB
# RISC-V 32-Bit Bit Manipulation Unit (BMU)
## Phase 2 Architecture & Functional Specification (Updated Design)

**Document Version:** 2.0  
**Target Architecture:** RV32 (32-bit RISC-V)  
**Supported Extensions:** Zba, Zbb, Zbs, Zbp (Complete Reference)  

---

## 1. Overview & Phase 2 Scope

In Phase 2, the **Bit Manipulation Unit (BMU)** is expanded to its full feature set. In addition to all instructions supported in Phase 1 (Zba, Zbb, Zbs), Phase 2 introduces:
1. **Halfword & Byte Packing (`PACK`, `PACKU`, `PACKH`):** High-throughput data packing and unpacking for multimedia, DSP, and memory-bandwidth optimization.
2. **Bitwise OR-Combine Byte (`ORC_B`):** Fast byte-level string scanning, null-terminator search, and delimiter detection.

Acting as RTL designer, you must:
- Fix all design defects identified during Phase 1.
- Implement these four new instructions into the RTL design.
- Update your UVM testbench environment to verify the complete, defect-free unit.

---

## 2. Hardware Interface & Port Definitions

The interface remains identical to Phase 1. The top-level module is `bmu_top`. All signals are synchronous to `clk` with an active-low asynchronous reset `rst_n`.

```
                           +----------------------+
                           |       bmu_top        |
             clk --------->|                      |
           rst_n --------->|                      |
                           |                      |
         valid_i --------->|                      |----------> valid_o
      op_i [5:0] --------->|                      |----------> rd_o [31:0]
    rs1_i [31:0] --------->|                      |
    rs2_i [31:0] --------->|                      |
                           +----------------------+
```

### 2.1 Interface Signals

| Signal | Direction | Width | Description |
| :--- | :---: | :---: | :--- |
| `clk` | Input | 1 | Master system clock (rising-edge active). |
| `rst_n` | Input | 1 | Asynchronous active-low reset. |
| `valid_i` | Input | 1 | Input transaction valid strobe. Samples `op_i`, `rs1_i`, `rs2_i` on rising clock edge. |
| `op_i` | Input | 6 | Operation selector enum (`bmu_pkg::bmu_op_e`). |
| `rs1_i` | Input | 32 | Primary 32-bit input operand (RS1). |
| `rs2_i` | Input | 32 | Secondary 32-bit input operand (RS2). For shift/rotate/single-bit operations, bits `[4:0]` specify shift/index. |
| `valid_o` | Output | 1 | Output valid flag. Asserted exactly 1 cycle after `valid_i = 1`. |
| `rd_o` | Output | 32 | Destination output data register. Captures result 1 cycle after `valid_i = 1`. **When `valid_i = 0`, `rd_o` must hold its previous value.** |

---

## 3. Timing & Protocol Specification

- **Pipeline Latency:** Fixed **1 clock cycle** latency.
- **Throughput:** Maximum rate of **1 operation per cycle** (back-to-back streaming supported).
- **Reset State:** When `rst_n == 0`, `valid_o` is driven to `0` and `rd_o` is cleared to `32'h0000_0000`.
- **Hold State:** When `valid_i == 0`, `valid_o` drops to `0` on the following cycle, and `rd_o` **must retain** its last valid computation.

---

## 4. Complete Phase 2 Operation Encodings (`bmu_pkg::bmu_op_e`)

The enum `bmu_op_e` in `bmu_pkg.sv` is expanded to include the new Phase 2 instructions:

| Enum Identifier | Opcode Value | Extension | Category | Status |
| :--- | :---: | :--- | :--- | :--- |
| `BMU_SH1ADD` | `6'd0` | Zba | Address Generation | Phase 1 Base |
| `BMU_SH2ADD` | `6'd1` | Zba | Address Generation | Phase 1 Base |
| `BMU_SH3ADD` | `6'd2` | Zba | Address Generation | Phase 1 Base (Must be bug-free) |
| `BMU_ANDN`   | `6'd3` | Zbb | Logic with Invert | Phase 1 Base |
| `BMU_ORN`    | `6'd4` | Zbb | Logic with Invert | Phase 1 Base |
| `BMU_XNOR`   | `6'd5` | Zbb | Logic with Invert | Phase 1 Base |
| `BMU_CLZ`    | `6'd6` | Zbb | Bit Counter | Phase 1 Base (Must be bug-free) |
| `BMU_CTZ`    | `6'd7` | Zbb | Bit Counter | Phase 1 Base (Must be bug-free) |
| `BMU_CPOP`   | `6'd8` | Zbb | Bit Counter | Phase 1 Base |
| `BMU_SEXT_B` | `6'd9` | Zbb | Sign-Extension | Phase 1 Base (Must be bug-free) |
| `BMU_SEXT_H` | `6'd10` | Zbb | Sign-Extension | Phase 1 Base |
| `BMU_MIN`    | `6'd11` | Zbb | Relational Min/Max | Phase 1 Base (Must be bug-free) |
| `BMU_MAX`    | `6'd12` | Zbb | Relational Min/Max | Phase 1 Base |
| `BMU_MINU`   | `6'd13` | Zbb | Relational Min/Max | Phase 1 Base |
| `BMU_MAXU`   | `6'd14` | Zbb | Relational Min/Max | Phase 1 Base |
| `BMU_ROL`    | `6'd15` | Zbb | Barrel Shifter | Phase 1 Base |
| `BMU_ROR`    | `6'd16` | Zbb | Barrel Shifter | Phase 1 Base |
| `BMU_REV8`   | `6'd17` | Zbb | Byte Permutation | Phase 1 Base |
| **`BMU_ORC_B`**| **`6'd18`** | **Zbb** | **Byte Permutation** | **NEW FEATURE (Phase 2)** |
| `BMU_BSET`   | `6'd19` | Zbs | Single-Bit | Phase 1 Base |
| `BMU_BCLR`   | `6'd20` | Zbs | Single-Bit | Phase 1 Base (Must be bug-free) |
| `BMU_BINV`   | `6'd21` | Zbs | Single-Bit | Phase 1 Base |
| `BMU_BEXT`   | `6'd22` | Zbs | Single-Bit | Phase 1 Base (Must be bug-free) |
| **`BMU_PACK`** | **`6'd23`** | **Zbp** | **Data Packing** | **NEW FEATURE (Phase 2)** |
| **`BMU_PACKU`**| **`6'd24`** | **Zbp** | **Data Packing** | **NEW FEATURE (Phase 2)** |
| **`BMU_PACKH`**| **`6'd25`** | **Zbp** | **Data Packing** | **NEW FEATURE (Phase 2)** |
| `BMU_NOP`    | `6'd63` | Control | NOP / Idle | Returns `32'h0000_0000` |

---

## 5. Functional Specification for New Phase 2 Instructions

### 5.1 `BMU_ORC_B` (Bitwise OR-Combine across Bytes)
- **Extension:** Zbb
- **Description:** For each of the four 8-bit bytes in `rs1`, if any bit in that byte is non-zero, set all 8 bits of that byte to `1` (`0xFF`). If all 8 bits are zero, the byte remains `0x00`.
- **Bit-Level Definition:**
  - `rd[7:0]   = (|rs1[7:0])   ? 8'hFF : 8'h00`
  - `rd[15:8]  = (|rs1[15:8])  ? 8'hFF : 8'h00`
  - `rd[23:16] = (|rs1[23:16]) ? 8'hFF : 8'h00`
  - `rd[31:24] = (|rs1[31:24]) ? 8'hFF : 8'h00`
- **Example:**
  - `rs1 = 32'h0012_0034` -> `rd = 32'h00FF_00FF`
  - `rs1 = 32'h0000_0000` -> `rd = 32'h0000_0000`
  - `rs1 = 32'h8001_0204` -> `rd = 32'hFFFF_FFFF`

---

### 5.2 `BMU_PACK` (Pack Low Halfwords)
- **Extension:** Zbp
- **Description:** Combines the lower 16-bit halfword of `rs1` and the lower 16-bit halfword of `rs2` into a single 32-bit word.
- **Bit-Level Definition:**
  - `rd[15:0]  = rs1[15:0]`
  - `rd[31:16] = rs2[15:0]`
- **Example:**
  - `rs1 = 32'hAAAA_1234`, `rs2 = 32'hBBBB_5678`
  - `rd = 32'h5678_1234`

---

### 5.3 `BMU_PACKU` (Pack High Halfwords)
- **Extension:** Zbp
- **Description:** Combines the upper 16-bit halfword of `rs1` and the upper 16-bit halfword of `rs2` into a single 32-bit word.
- **Bit-Level Definition:**
  - `rd[15:0]  = rs1[31:16]`
  - `rd[31:16] = rs2[31:16]`
- **Example:**
  - `rs1 = 32'hAAAA_1234`, `rs2 = 32'hBBBB_5678`
  - `rd = 32'hBBBB_AAAA`

---

### 5.4 `BMU_PACKH` (Pack Low Bytes)
- **Extension:** Zbp
- **Description:** Packs the lowest byte of `rs1` into `rd[7:0]` and the lowest byte of `rs2` into `rd[15:8]`, with the upper 16 bits zero-extended.
- **Bit-Level Definition:**
  - `rd[7:0]   = rs1[7:0]`
  - `rd[15:8]  = rs2[7:0]`
  - `rd[31:16] = 16'h0000`
- **Example:**
  - `rs1 = 32'hAAAA_BB12`, `rs2 = 32'hCCCC_DD34`
  - `rd = 32'h0000_3412`

---

## 6. Functional Summary of All Operations (Regression Target)

For complete mathematical definitions and corner-case rules of the baseline Phase 1 operations (e.g. `CLZ(0) = 32`, `CTZ(MSB) = 31`, `MIN` signed polarity, `SH3ADD` scale of 8, etc.), refer to the Phase 1 specification. In Phase 2, **all 26 operations** must pass with 100% compliance in simulation.
