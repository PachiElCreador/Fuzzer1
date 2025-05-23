# Dataset Overview

---

This fuzzing system performs both ISA and RTL simulations of a RISC-V processor design, executing hundreds or thousands of iterations to detect mismatches and potential bugs. For each iteration, the system collects and stores the values of relevant input and output signals, as well as the processor's program counter, current instruction, and coverage metrics. This enables detailed post-simulation analysis and facilitates the development of new bug-detection and assertion-generation techniques.

---

## **How the Values Were Obtained**

### a) **Execution and Simulation**

- The main script (`Fuzzer.py`) controls the entire fuzzing workflow.
- For each iteration:
    1. Input vectors are generated and applied to the processor core.
    2. Both the **ISA** and **RTL** models are simulated.
    3. Timeouts, assertion failures, mismatches, and coverage changes are detected and recorded.

### b) **Signal Extraction and Logging**

- During each RTL simulation cycle, the system extracts the current value of:
    - **PC** (program counter)
    - **Instruction** being executed
    - **Coverage** counter
    - **Input and output port signals** of the DUT, these where obtained by the design verilog file.
- These values are captured using a logger, which appends each row to a pandas DataFrame and saves the full dataset at the end of each iteration.

### c) **Formatting and Output**

- Each row in the datasets represents a single cycle of RTL simulation, with the value of every relevant signal for that cycle.
- All values are recorded as hexadecimal strings or integers.

---

## 3. **Field Descriptions**

Below is a brief description of each column in the dataset:

| Field | Description |
| --- | --- |
| `cycle` | Simulation cycle number (starts at 0; decimal) |
| `pc` | Program Counter (PC) at this cycle  |
| `inst` | Current instruction opcode being executed |
| `coverage` | Current cumulative coverage value (decimal) |
| `clock` | DUT clock signal (stays at one because only appends RisingEdge) |
| `reset` | DUT reset signal  |

### Input Signals

1. **clock**
2. **reset**
3. auto_intsink_in_sync_0
4. auto_int_in_xing_in_2_sync_0
5. auto_int_in_xing_in_1_sync_0
6. auto_int_in_xing_in_0_sync_0
7. auto_int_in_xing_in_0_sync_1
8. auto_tl_master_xing_out_a_ready
9. auto_tl_master_xing_out_b_valid
10. auto_tl_master_xing_out_b_bits_opcode
11. auto_tl_master_xing_out_b_bits_param
12. auto_tl_master_xing_out_b_bits_size
13. auto_tl_master_xing_out_b_bits_source
14. **auto_tl_master_xing_out_b_bits_address**
15. auto_tl_master_xing_out_b_bits_mask
16. **auto_tl_master_xing_out_d_bits_data**
17. auto_tl_master_xing_out_b_bits_corrupt
18. auto_tl_master_xing_out_c_ready
19. **auto_tl_master_xing_out_d_valid**
20. **auto_tl_master_xing_out_d_bits_opcode**
21. **auto_tl_master_xing_out_d_bits_param**
22. **auto_tl_master_xing_out_d_bits_size**
23. **auto_tl_master_xing_out_d_bits_source**
24. **auto_tl_master_xing_out_d_bits_sink**
25. auto_tl_master_xing_out_d_bits_denied
26. auto_tl_master_xing_out_d_bits_data
27. auto_tl_master_xing_out_d_bits_corrupt
28. auto_tl_master_xing_out_e_ready
29. constants_hartid
30. constants_reset_vector
31. metaReset
32. intsink_1_halt
33. frontend_halt
34. tlMasterXbar_halt
35. intsink_halt
36. ptw_halt
37. core_halt
38. lsu_halt
39. dcache_halt
40. intXbar_halt
41. intsink_2_halt
42. buffer_halt
43. trace_halt
44. intsink_3_halt
45. hellaCacheArb_halt

---

### Output Signals

1. auto_trace_out_0_clock
2. auto_trace_out_0_reset
3. auto_trace_out_0_valid
4. auto_trace_out_0_iaddr
5. auto_trace_out_0_insn
6. auto_trace_out_0_priv
7. auto_trace_out_0_exception
8. auto_trace_out_0_interrupt
9. auto_trace_out_0_cause
10. auto_trace_out_0_tval
11. **auto_tl_master_xing_out_a_valid**
12. **auto_tl_master_xing_out_a_bits_opcode**
13. **auto_tl_master_xing_out_a_bits_param**
14. **auto_tl_master_xing_out_a_bits_size**
15. **auto_tl_master_xing_out_a_bits_source**
16. **auto_tl_master_xing_out_a_bits_address**
17. auto_tl_master_xing_out_a_bits_mask
18. auto_tl_master_xing_out_a_bits_data
19. auto_tl_master_xing_out_a_bits_corrupt
20. auto_tl_master_xing_out_b_ready
21. **auto_tl_master_xing_out_c_valid**
22. **auto_tl_master_xing_out_c_bits_opcode**
23. auto_tl_master_xing_out_c_bits_param
24. auto_tl_master_xing_out_c_bits_size
25. **auto_tl_master_xing_out_c_bits_source**
26. **auto_tl_master_xing_out_c_bits_address**
27. **auto_tl_master_xing_out_c_bits_data**
28. auto_tl_master_xing_out_c_bits_corrupt
29. auto_tl_master_xing_out_d_ready
30. **auto_tl_master_xing_out_e_valid**
31. **auto_tl_master_xing_out_e_bits_sink**
32. auto_wfi_out_0
33. auto_cease_out_0
34. auto_halt_out_0
35. **io_covSum**
36. metaAssert

---

**Notes:**

- Signals in **bold** are those that actually change during execution (are not always zero or constant).
- All signals with names such as `auto_tl_master_xing_out_*` correspond to TileLink bus signals used for communication with memory or peripherals.
- Most signals will remain zero unless there is a specific transaction or event on that channel.

In particular, the `auto_tl_master_xing_out_a*` signals and `auto_tl_master_xing_out_d_bits_data` are the most relevant.

### **`auto_tl_master_xing_out_a*`**

- These correspond to the Channel A signals of the TileLink protocol from the master (the core) to the slave (memory or peripherals).
- Channel A is primarily used for memory access requests: reads, writes, and atomic operations.

### **`auto_tl_master_xing_out_d_bits_data`**

- This signal is part of Channel D of TileLink, which is used to respond to transactions initiated on Channel A.
- `_d_bits_data` carries the response data back to the core.

- *A (auto_tl_master_xing_out_a*)**: Request from the core → memory/peripheral
- *D (auto_tl_master_xing_out_d*)**: Response from memory/peripheral → core

---

### **ISA State Columns (per-cycle, matched by PC and instruction):**

| Field | Description |
| --- | --- |
| `x_regs_isa` | Dictionary with the value of each RISC-V integer register (`x0`–`x31`) as seen by the ISA simulator at this PC/instruction. |
| `f_regs_isa` | Dictionary with the value of each RISC-V floating-point register (`f0`–`f31`) as seen by the ISA simulator at this PC/instruction. |
| `csrs_isa` | Dictionary with the value of each RISC-V CSR (e.g., `mstatus`, `mepc`, `mcause`, etc.) at this PC/instruction. |
| `mem_isa` | Dictionary with memory access fields from the ISA simulator for this instruction: includes `mem_rd_addr`, `mem_rd_val`, `mem_wr_addr`, and `mem_wr_val`. |

These columns allow you to compare, for every cycle and instruction, the internal architectural state between the RTL design and the reference ISA simulation, supporting detailed bug and divergence analysis.

---

## 4. **Example Row**

Each value below is represented in binary with the column width chosen to fit the maximum observed bit length per signal.

```
cycle | pc                                       | inst                             | coverage | clock | reset | auto_intsink_in_sync_0 | auto_int_in_xing_in_2_sync_0 | auto_int_in_xing_in_1_sync_0 | auto_int_in_xing_in_0_sync_0 | auto_int_in_xing_in_0_sync_1 | auto_tl_master_xing_out_a_ready | auto_tl_master_xing_out_b_valid | auto_tl_master_xing_out_b_bits_opcode | auto_tl_master_xing_out_b_bits_param | auto_tl_master_xing_out_b_bits_size | auto_tl_master_xing_out_b_bits_source | auto_tl_master_xing_out_b_bits_address | auto_tl_master_xing_out_b_bits_mask | auto_tl_master_xing_out_b_bits_data | auto_tl_master_xing_out_b_bits_corrupt | auto_tl_master_xing_out_c_ready | auto_tl_master_xing_out_d_valid | auto_tl_master_xing_out_d_bits_opcode | auto_tl_master_xing_out_d_bits_param | auto_tl_master_xing_out_d_bits_size | auto_tl_master_xing_out_d_bits_source | auto_tl_master_xing_out_d_bits_sink | auto_tl_master_xing_out_d_bits_denied | auto_tl_master_xing_out_d_bits_data                              | auto_tl_master_xing_out_d_bits_corrupt | auto_tl_master_xing_out_e_ready | constants_hartid | constants_reset_vector | metaReset | intsink_1_halt | frontend_halt | tlMasterXbar_halt | intsink_halt | ptw_halt | core_halt | lsu_halt | dcache_halt | intXbar_halt | intsink_2_halt | buffer_halt | trace_halt | intsink_3_halt | hellaCacheArb_halt | auto_trace_out_0_clock | auto_trace_out_0_reset | auto_trace_out_0_valid | auto_trace_out_0_iaddr | auto_trace_out_0_insn | auto_trace_out_0_priv | auto_trace_out_0_exception | auto_trace_out_0_interrupt | auto_trace_out_0_cause | auto_trace_out_0_tval | auto_tl_master_xing_out_a_valid | auto_tl_master_xing_out_a_bits_opcode | auto_tl_master_xing_out_a_bits_param | auto_tl_master_xing_out_a_bits_size | auto_tl_master_xing_out_a_bits_source | auto_tl_master_xing_out_a_bits_address | auto_tl_master_xing_out_a_bits_mask | auto_tl_master_xing_out_a_bits_data | auto_tl_master_xing_out_a_bits_corrupt | auto_tl_master_xing_out_b_ready | auto_tl_master_xing_out_c_valid | auto_tl_master_xing_out_c_bits_opcode | auto_tl_master_xing_out_c_bits_param | auto_tl_master_xing_out_c_bits_size | auto_tl_master_xing_out_c_bits_source | auto_tl_master_xing_out_c_bits_address | auto_tl_master_xing_out_c_bits_data                              | auto_tl_master_xing_out_c_bits_corrupt | auto_tl_master_xing_out_d_ready | auto_tl_master_xing_out_e_valid | auto_tl_master_xing_out_e_bits_sink | auto_wfi_out_0 | auto_cease_out_0 | auto_halt_out_0 | io_covSum      | metaAssert
00027 | 0000000000000000000000010000000000000000 | 00000000000000000000001010010111 | 00000138 | 00001 | 00000 | 0000000000000000000000 | 0000000000000000000000000000 | 0000000000000000000000000000 | 0000000000000000000000000000 | 0000000000000000000000000000 | 0000000000000000000000000000001 | 0000000000000000000000000000000 | 0000000000000000000000000000000000000 | 000000000000000000000000000000000000 | 00000000000000000000000000000000000 | 0000000000000000000000000000000000000 | 00000000000000000000000000000000000000 | 00000000000000000000000000000000000 | 00000000000000000000000000000000000 | 00000000000000000000000000000000000000 | 0000000000000000000000000000001 | 0000000000000000000000000000000 | 0000000000000000000000000000000000000 | 000000000000000000000000000000000000 | 00000000000000000000000000000000000 | 0000000000000000000000000000000000000 | 00000000000000000000000000000000000 | 0000000000000000000000000000000000000 | 0000000000000000000000000000000000000000000000000000000000000000 | 00000000000000000000000000000000000000 | 0000000000000000000000000000001 | 0000000000000000 | 0000010000000000000000 | 000000000 | 00000000000000 | 0000000000000 | 00000000000000000 | 000000000000 | 00000000 | 000000000 | 00000000 | 00000000000 | 000000000000 | 00000000000000 | 00000000000 | 0000000000 | 00000000000000 | 000000000000000000 | 0000000000000000000000 | 0000000000000000000000 | 0000000000000000000000 | 0000000000000000000000 | 000000000000000000000 | 000000000000000000000 | 00000000000000000000000000 | 00000000000000000000000000 | 0000000000000000000000 | 000000000000000000000 | 0000000000000000000000000000000 | 0000000000000000000000000000000000000 | 000000000000000000000000000000000000 | 00000000000000000000000000000000000 | 0000000000000000000000000000000000000 | 00000000000000000000000000000000000000 | 00000000000000000000000000000000000 | 00000000000000000000000000000000000 | 00000000000000000000000000000000000000 | 0000000000000000000000000000001 | 0000000000000000000000000000000 | 0000000000000000000000000000000000000 | 000000000000000000000000000000000000 | 00000000000000000000000000000000000 | 0000000000000000000000000000000000000 | 00000000000000000000000000000000000000 | 0000000000000000000000000000000000000000000000000000000000000000 | 00000000000000000000000000000000000000 | 0000000000000000000000000000001 | 0000000000000000000000000000000 | 00000000000000000000000000000000000 | 00000000000000 | 0000000000000000 | 000000000000000 | 00000010001010 | 0000000000
...
00033 | 0000000000000000000000010000000000000100 | 00000010000000101000010110010011 | 00000149 | 00001 | 00000 | 0000000000000000000000 | 0000000000000000000000000000 | 0000000000000000000000000000 | 0000000000000000000000000000 | 0000000000000000000000000000 | 0000000000000000000000000000001 | 0000000000000000000000000000000 | 0000000000000000000000000000000000000 | 000000000000000000000000000000000000 | 00000000000000000000000000000000000 | 0000000000000000000000000000000000000 | 00000000000000000000000000000000000000 | 00000000000000000000000000000000000 | 00000000000000000000000000000000000 | 00000000000000000000000000000000000000 | 0000000000000000000000000000001 | 0000000000000000000000000000000 | 0000000000000000000000000000000000000 | 000000000000000000000000000000000000 | 00000000000000000000000000000000000 | 0000000000000000000000000000000000000 | 00000000000000000000000000000000000 | 0000000000000000000000000000000000000 | 0000000000000000000000000000000000000000000000000000000000000000 | 00000000000000000000000000000000000000 | 0000000000000000000000000000001 | 0000000000000000 | 0000010000000000000000 | 000000000 | 00000000000000 | 0000000000000 | 00000000000000000 | 000000000000 | 00000000 | 000000000 | 00000000 | 00000000000 | 000000000000 | 00000000000000 | 00000000000 | 0000000000 | 00000000000000 | 000000000000000000 | 0000000000000000000000 | 0000000000000000000000 | 0000000000000000000000 | 0000000000000000000000 | 000000000000000000000 | 000000000000000000000 | 00000000000000000000000000 | 00000000000000000000000000 | 0000000000000000000000 | 000000000000000000000 | 0000000000000000000000000000000 | 0000000000000000000000000000000000000 | 000000000000000000000000000000000000 | 00000000000000000000000000000000000 | 0000000000000000000000000000000000000 | 00000000000000000000000000000000000000 | 00000000000000000000000000000000000 | 00000000000000000000000000000000000 | 00000000000000000000000000000000000000 | 0000000000000000000000000000001 | 0000000000000000000000000000000 | 0000000000000000000000000000000000000 | 000000000000000000000000000000000000 | 00000000000000000000000000000000000 | 0000000000000000000000000000000000000 | 00000000000000000000000000000000000000 | 0000000000000000000000000000000000000000000000000000000000000000 | 00000000000000000000000000000000000000 | 0000000000000000000000000000001 | 0000000000000000000000000000000 | 00000000000000000000000000000000000 | 00000000000000 | 0000000000000000 | 000000000000000 | 00000010010101 | 0000000000
...
00041 | 0000000000000000000000010000000000001000 | 11110001010000000010010101110011 | 00000152 | 00001 | 00000 | 0000000000000000000000 | 0000000000000000000000000000 | 0000000000000000000000000000 | 0000000000000000000000000000 | 0000000000000000000000000000 | 0000000000000000000000000000001 | 0000000000000000000000000000000 | 0000000000000000000000000000000000000 | 000000000000000000000000000000000000 | 00000000000000000000000000000000000 | 0000000000000000000000000000000000000 | 00000000000000000000000000000000000000 | 00000000000000000000000000000000000 | 00000000000000000000000000000000000 | 00000000000000000000000000000000000000 | 0000000000000000000000000000001 | 0000000000000000000000000000000 | 0000000000000000000000000000000000000 | 000000000000000000000000000000000000 | 00000000000000000000000000000000000 | 0000000000000000000000000000000000000 | 00000000000000000000000000000000000 | 0000000000000000000000000000000000000 | 0000000000000000000000000000000000000000000000000000000000000000 | 00000000000000000000000000000000000000 | 0000000000000000000000000000001 | 0000000000000000 | 0000010000000000000000 | 000000000 | 00000000000000 | 0000000000000 | 00000000000000000 | 000000000000 | 00000000 | 000000000 | 00000000 | 00000000000 | 000000000000 | 00000000000000 | 00000000000 | 0000000000 | 00000000000000 | 000000000000000000 | 0000000000000000000000 | 0000000000000000000000 | 0000000000000000000000 | 0000000000000000000000 | 000000000000000000000 | 000000000000000000000 | 00000000000000000000000000 | 00000000000000000000000000 | 0000000000000000000000 | 000000000000000000000 | 0000000000000000000000000000000 | 0000000000000000000000000000000000000 | 000000000000000000000000000000000000 | 00000000000000000000000000000000000 | 0000000000000000000000000000000000000 | 00000000000000000000000000000000000000 | 00000000000000000000000000000000000 | 00000000000000000000000000000000000 | 00000000000000000000000000000000000000 | 0000000000000000000000000000001 | 0000000000000000000000000000000 | 0000000000000000000000000000000000000 | 000000000000000000000000000000000000 | 00000000000000000000000000000000000 | 0000000000000000000000000000000000000 | 00000000000000000000000000000000000000 | 0000000000000000000000000000000000000000000000000000000000000000 | 00000000000000000000000000000000000000 | 0000000000000000000000000000001 | 0000000000000000000000000000000 | 00000000000000000000000000000000000 | 00000000000000 | 0000000000000000 | 000000000000000 | 00000010011000 | 0000000000

```

---
