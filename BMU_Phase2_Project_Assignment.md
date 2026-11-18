> # ENCS5131, HARDWARE DESIGN LAB
## Phase 2: RTL Bug Fixing, Feature Expansion & Regression Closure

**Target DUT:** RISC-V 32-Bit Bit Manipulation Unit (`bmu_top`)  
**Role:** RTL Design & Integration Engineer  
**Methodology:** SystemVerilog RTL Design + UVM Regression Verification  
**Deliverables:** Fixed & Expanded RTL, Updated UVM Environment, Final Verification Report  

---

## 1. Project Overview & Objectives

In Phase 1, you acted as a Verification Engineer and identified multiple functional and architectural defects in the BMU design.

In **Phase 2**, you will switch roles and put on the **RTL Designer Hat**. You are now responsible for:
1. **Fixing all RTL bugs** you identified during Phase 1.
2. **Expanding the RTL** to implement 4 brand-new instructions specified in [BMU_Phase2_Specification.md](BMU_Phase2_Specification.md):
   - `BMU_ORC_B` (Bitwise OR-Combine across bytes)
   - `BMU_PACK` (Pack lower 16 bits of rs1 and rs2)
   - `BMU_PACKU` (Pack upper 16 bits of rs1 and rs2)
   - `BMU_PACKH` (Pack byte 0 of rs1 and rs2)
3. **Updating your UVM Verification Environment** to thoroughly verify the new features.
4. **Achieving 100% test pass rate and > 95% functional coverage closure.**

---

## 2. Tasks & Required Deliverables

### Task 1: RTL Bug Fixing
Using your Phase 1 Bug Report as your punch list:
1. Modify the RTL submodules in your design directory to fix every identified defect.
2. Ensure your fixes adhere to clean, synthesizable SystemVerilog practices.
3. Verify that your fixes do not introduce any syntax errors or lint warnings.

### Task 2: RTL Feature Expansion (New Instructions)
Implement the four new instructions defined in Section 5 of [BMU_Phase2_Specification.md](BMU_Phase2_Specification.md):
1. **`bmu_pkg.sv`:** Declare the new enum values:
   - `BMU_ORC_B = 6'd18`
   - `BMU_PACK  = 6'd23`
   - `BMU_PACKU = 6'd24`
   - `BMU_PACKH = 6'd25`
2. **`bmu_bitwise_permute.sv`:** Implement the combinational logic for the four new instructions.
3. **`bmu_top.sv`:** Ensure the result multiplexer routes the output of `bmu_bitwise_permute` to `res_comb` for these new opcodes.

### Task 3: UVM Testbench Updates
Update your Phase 1 UVM verification environment:
1. **Transaction Items & Sequences:** Extend `bmu_seq_item.sv` constraints to generate transactions for the four new opcodes with balanced distribution.
2. **Scoreboard Reference Model:** Update the golden predictor inside `bmu_scoreboard.sv` with reference models for `ORC_B`, `PACK`, `PACKU`, and `PACKH`.
3. **Functional Coverage:** Add new coverpoints and cross-coverage bins for the new instructions and their operand combinations.

### Task 4: Sanity Check & Verification Reference
Before launching full regression:
- Run the test stimulus provided in [phase2_expected_new_features.txt](phase2_expected_new_features.txt).
- Confirm that both your new feature implementations and your bug fixes produce the exact expected outputs.

### Task 5: Final Regression & Coverage Closure
Execute a full regression suite (minimum 5,000 randomized transactions plus directed tests):
- **0 Scoreboard Errors:** All transactions must match the golden reference model.
- **Opcode Coverage:** 100% of all 26 operations in `bmu_op_e` exercised.
- **Shift & Index Coverage:** 100% coverage across all 32 bit-indices/shift values.
- **Overall Functional Coverage:** **> 95%** total functional coverage achieved.

---

## 3. Final Report Deliverables

Your Phase 2 submission must include:
1. **RTL Design Summary:**
   - Explanation of how each bug was fixed (including before/after code snippets).
   - Architectural implementation details of the 4 new instructions.
2. **RTL Source Code:**
   - Clean, bug-free, expanded SystemVerilog files (`bmu_pkg.sv`, `bmu_top.sv`, and submodules).
3. **Updated UVM Environment:**
   - Complete updated UVM testbench files.
4. **Functional Coverage Report:**
   - Coverage report summary demonstrating > 95% coverage closure.
5. **Simulation Transcript:**
   - Log showing 100% passed tests with zero mismatches.

---

## 4. Grading (100 Points Total)

| Category | Points | Description |
| :--- | :---: | :--- |
| **Bug Fixing Quality** | 25 | All Phase 1 bugs correctly resolved in RTL without side effects. |
| **New RTL Implementation** | 25 | Correct, synthesizable, modular implementation of `ORC_B`, `PACK`, `PACKU`, `PACKH`. |
| **UVM Environment Updates** | 20 | Scoreboard predictor, sequences, and items properly extended for new features. |
| **Functional Coverage Closure** | 20 | > 95% functional coverage achieved with clean, meaningful covergroups. |
| **Final Documentation & Cleanliness** | 10 | High-quality engineering report, clean directory structure, zero simulation warnings. |
