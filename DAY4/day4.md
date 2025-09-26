# Day 4: Gate-Level Simulation (GLS), Blocking vs Non-Blocking in Verilog, and Synthesis-Simulation Mismatch

Welcome to Day 4! This session covers three fundamental topics crucial for digital design verification and synthesis quality:

- Gate-Level Simulation (GLS)
- Blocking vs Non-Blocking Assignments in Verilog
- Synthesis-Simulation Mismatch

---

## 1. Gate-Level Simulation (GLS)

Gate-Level Simulation is performed post-synthesis using the gate-level netlist to verify functional correctness and timing behavior. It can include realistic timing delays through SDF annotations for setup/hold verification and power estimation.

- Validates synthesized RTL mapped to gates.
- Detects timing violations and glitches early.
- Verifies testability including scan chains before layout.

GLS happens after synthesis and before physical design.

---

## 2. Synthesis-Simulation Mismatch

Simulation and synthesis mismatch occurs when RTL simulation results differ from gate-level or hardware behavior. Common causes:

- Coding styles non-synthesizable by tools (delays, initial blocks).
- Missing or wrong sensitivity lists in always blocks.
- Tool differences in interpreting RTL constructs.
- Ambiguous or incomplete RTL (missing else branches).

Mitigation involves writing clean, unambiguous synthesizable RTL using best practices.

---

## 3. Blocking vs Non-Blocking Assignments in Verilog

### 3.1 Blocking Assignments (`=`)

- Execute sequentially and immediately.
- Suitable for combinational logic.
- Updates happen in code order.

Example:
always @(*) y = a & b;

text

### 3.2 Non-Blocking Assignments (`<=`)

- Schedule updates at the end of the time step.
- Suitable for sequential logic.
- Updates concurrent, modeling register behavior.

Example:
always @(posedge clk) q <= d;

text

### 3.3 Comparison Table

| Blocking (`=`)         | Non-Blocking (`<=`)        |
|-----------------------|----------------------------|
| Immediate execution    | Scheduled execution        |
| Used in combinational  | Used in sequential         |
| Updates in order      | Updates after all RHS eval |
| Can cause race issues  | Avoids race conditions     |

---

## 4. Labs

### Lab 1: Ternary Operator MUX

Example of 2:1 mux using ternary operator assigning `y = sel ? i1 : i0`.

Yosys commands:
read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
read_verilog ternary_operator_mux.v
synth -top ternary_operator_mux
dfflibmap -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
write_verilog -noattr ternary_operator_net.v
quit

text

Simulation commands:
iverilog -o ternary_operator_sim ../my_lib/verilog_model/primitives.v ../my_lib/verilog_model/sky130_fd_sc_hd.v ternary_operator_net.v tb_ternary_operator_mux.v
vvp ternary_operator_sim
gtkwave tb_ternary_operator_mux.vcd

text

*Insert simulation and synthesis screenshots here*

---

### Lab 2: Bad MUX Example

MUX with incomplete sensitivity list and wrong assignment styles, demonstrating pitfalls.

Yosys commands:
read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
read_verilog bad_mux.v
synth -top bad_mux
dfflibmap -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
write_verilog -noattr bad_mux_net.v
quit

text

Simulation commands:
iverilog -o bad_mux_sim ../my_lib/verilog_model/primitives.v ../my_lib/verilog_model/sky130_fd_sc_hd.v bad_mux_net.v tb_bad_mux.v
vvp bad_mux_sim
gtkwave tb_bad_mux.vcd

text

*Insert simulation and synthesis screenshots here*

---

## 5. Summary

- Gate-Level Simulation is essential for verifying synthesized designs under realistic conditions.
- Proper RTL coding prevents synthesis-simulation mismatches.
- Use blocking assignments in combinational and non-blocking in sequential code to model hardware accurately.
- Labs reinforced theoretical concepts through practical synthesis and simulation.

---
