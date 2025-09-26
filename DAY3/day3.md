
# Day 3: Combinational and Sequential Optimization

Welcome to Day 3 of this workshop! Today we discuss optimization of combinational and sequential circuits, introducing techniques to enhance efficiency and performance.

---

# Table of Contents

- [Introduction to Logic Optimization](#1-introduction-to-logic-optimization)
- [Combinational Logic Optimization](#2-combinational-logic-optimization)
  - [Why Optimize?](#why-optimize)
  - [Constant Propagation](#constant-propagation)
  - [Boolean Logic Optimization](#boolean-logic-optimization)
- [Sequential Logic Optimization](#3-sequential-logic-optimization)
  - [Why Optimize Sequential Logic?](#why-optimize-sequential-logic)
  - [Basic Sequential Constant Propagation](#basic-sequential-constant-propagation)
  - [Advanced Sequential Optimization Techniques](#advanced-sequential-optimization-techniques)
    - [State Optimization](#state-optimization)
    - [Cloning](#cloning)
    - [Retiming](#retiming)
- [Labs on Combinational Optimization](#4-labs-on-combinational-optimization)
  - [Lab 1: opt_check1.v](#lab-1-opt_check1v)
  - [Lab 2: opt_check2.v](#lab-2-opt_check2v)
  - [Lab 3: opt_check3.v](#lab-3-opt_check3v)
  - [Lab 4: opt_check4.v](#lab-4-opt_check4v)
  - [Lab 5: multiple_module_opt.v](#lab-5-multiple_module_optv)
- [Labs on Sequential Optimization](#5-labs-on-sequential-optimization)
  - [Lab 1: dff_const1.v](#lab-1-dff_const1v)
  - [Lab 2: dff_const2.v](#lab-2-dff_const2v)
  - [Lab 3: dff_const3.v](#lab-3-dff_const3v)
  - [Lab 4: dff_const4.v](#lab-4-dff_const4v)
  - [Lab 5: dff_const5.v](#lab-5-dff_const5v)
  - [Lab 6: counter_opt.v](#lab-6-counter_optv-unused-outputs)
  - [Lab 7: counter_opt2.v](#lab-7-counter_opt2v-unused-outputs)
- [Summary](#6-summary)
  
---

## 1. Introduction to Logic Optimization

Digital logic optimization targets reducing the complexity, area, and power consumption of digital circuits by simplifying both combinational and sequential logic. This is critical for designing efficient chips that meet stringent cost and performance requirements.

---

## 2. Combinational Logic Optimization

### Why Optimize?

- Reduced silicon area and gate count  
- Lower dynamic and static power  
- Improved circuit speed by shortening critical paths  

### Constant Propagation

Constant propagation is a direct optimization method where fixed signals (constants 0 or 1) are propagated through the logic to simplify or eliminate unnecessary gates.

**Example:**  
If input signal `A` = 0 always, expressions involving `A` simplify drastically, potentially collapsing complex circuits to simple inverters or even direct wires.

<div align="center">
  ![Constant Propagation Example](./images/constant_propagation_example.png)
</div>

### Boolean Logic Optimization

Methods such as Karnaugh Maps, Quine-McCluskey, and Boolean algebra simplify logic expressions to their minimal form, eliminating redundant terms.

**Example Boolean Expression Reduction:**  
`assign y = a ? (b ? c : (c ? a : 0)) : (!c)`  
can be reduced by Boolean algebra to a minimal form, reducing gates and delay.

<div align="center">
  ![Boolean Logic Optimization](./images/boolean_logic_optimization.png)
</div>

---

## 3. Sequential Logic Optimization

### Basic Sequential Constant Propagation

Similar to combinational optimization but applied over sequential circuits, considering memory elements (flip-flops). Constants propagated through sequential logic help reduce overall complexity.

### Advanced Sequential Optimizations

- **State Optimization:** Merging equivalent states, optimal state encoding, and logic minimization to reduce FSM complexity and power.  
- **Cloning:** Duplicating heavily loaded cells/modules to balance timing and improve performance, especially in floorplan-aware synthesis.  
- **Retiming:** Repositioning registers across combinational logic to minimize max path delay without altering functional behavior.

---

## 4. Labs on Optimization

### Lab 1: opt_check1.v

// Simple redundant OR gate optimization
module opt_check1 (input a, input b, output y);
assign y = a | (a & b);
endmodule

text

**Explanation:**  
Yosys synthesis will simplify the redundant OR-AND combination.

<div align="center">
  ![Lab 1 Output](./images/lab1_output.png)
</div>

---

### Lab 2: opt_check2.v

// Redundant NOT logic optimization
module opt_check2 (input a, input b, output y);
assign y = ~(~a & ~b);
endmodule

text

**Explanation:**  
Yosys reduces this to a simplified equivalence of `a | b`.

<div align="center">
  ![Lab 2 Output](./images/lab2_output.png)
</div>

---

### Lab 3: opt_check3.v

// Repeated signals for optimization
module opt_check3 (input a, input b, input c, output y);
assign y = (a & b) | (a & b & c);
endmodule

text

**Explanation:**  
Redundant term `(a & b & c)` can be simplified to `(a & b)` only.

<div align="center">
  ![Lab 3 Output](./images/lab3_output.png)
</div>

---

### Lab 4: opt_check4.v

// Optimization using distributive law
module opt_check4 (input a, input b, input c, output y);
assign y = (a & b) | (a & c);
endmodule

text

**Explanation:**  
Shows distributive property optimization.

<div align="center">
  ![Lab 4 Output](./images/lab4_output.png)
</div>

---

### Lab 5: multiple_module_opt.v

module sub_module1 (input a, input b, output y);
assign y = a & b;
endmodule

module sub_module2 (input x, input z, output w);
assign w = x | z;
endmodule

module multiple_modules (input a, input b, input c, output y);
wire n1, n2;
sub_module1 u1 (.a(a), .b(b), .y(n1));
sub_module2 u2 (.x(n1), .z(c), .w(n2));
assign y = n2 & b;
endmodule

text

**Explanation:**  
Modular design showcasing synthesis optimization and flattening.

<div align="center">
  ![Lab 5 Output](./images/lab5_output.png)
</div>

---

## 3. Sequential Logic Optimization

Sequential logic circuits differ from combinational circuits by incorporating memory elements such as flip-flops. Their outputs depend not only on current inputs but also on the history of inputs stored in these sequential elements. This allows the implementation of state machines, counters, timers, and other complex synchronous systems.

### Why Optimize Sequential Logic?

- Minimize the total number of flip-flops and associated combinational logic.
- Reduce clock load to save dynamic power.
- Improve timing by retiming registers and balancing propagation delays.
- Simplify state machines via state reduction and optimal encoding.
- Replicate critical sequential elements (cloning) to improve timing and floorplan efficiency.

### Core Techniques Covered:

- Basic Sequential Constant Propagation  
- State Optimization  
- Sequential Logic Cloning  
- Retiming

---

## 5. Labs on Sequential Optimization

### Lab 1: dff_const1.v

**Description:**  
A D flip-flop with asynchronous reset that sets the output to 0, otherwise loads constant 1 on clock edge.

**Simulation Commands:**

iverilog -o dff_const1_out dff_const1.v tb_dff_const1.v
vvp dff_const1_out
gtkwave tb_dff_const1.vcd

text

**Synthesis Commands:**

read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
read_verilog dff_const1.v
synth -top dff_const1
dfflibmap -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib

text

**Simulation Waveform Screenshot:**  
![Insert simulation waveform image here for dff_const1]

**Synthesis Result Screenshot:**  
![Insert synthesis output/netlist image here for dff_const1]

---

### Lab 2: dff_const2.v

**Description:**  
A D flip-flop that always sets output to 1 regardless of reset.

**Simulation Commands:**

iverilog -o dff_const2_out dff_const2.v tb_dff_const2.v
vvp dff_const2_out
gtkwave tb_dff_const2.vcd

text

**Synthesis Commands:**

read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
read_verilog dff_const2.v
synth -top dff_const2
dfflibmap -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib

text

**Simulation Waveform Screenshot:**  
![Insert simulation waveform image here for dff_const2]

**Synthesis Result Screenshot:**  
![Insert synthesis output/netlist image here for dff_const2]

---

### Lab 3: dff_const3.v

**Description:**  
[Add short description for dff_const3]

**Simulation Commands:**

iverilog -o dff_const3_out dff_const3.v tb_dff_const3.v
vvp dff_const3_out
gtkwave tb_dff_const3.vcd

text

**Synthesis Commands:**  
read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
read_verilog dff_const3.v
synth -top dff_const3
dfflibmap -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib

text

**Simulation Waveform Screenshot:**  
![Insert simulation waveform image here for dff_const3]

**Synthesis Result Screenshot:**  
![Insert synthesis output/netlist image here for dff_const3]

---

### Lab 4: dff_const4.v

**Description:**  
[Add short description for dff_const4]

**Simulation Commands:**

iverilog -o dff_const4_out dff_const4.v tb_dff_const4.v
vvp dff_const4_out
gtkwave tb_dff_const4.vcd

text

**Synthesis Commands:**  
read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
read_verilog dff_const4.v
synth -top dff_const4
dfflibmap -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib

text

**Simulation Waveform Screenshot:**  
![Insert simulation waveform image here for dff_const4]

**Synthesis Result Screenshot:**  
![Insert synthesis output/netlist image here for dff_const4]

---

### Lab 5: dff_const5.v

**Description:**  
[Add short description for dff_const5]

**Simulation Commands:**

iverilog -o dff_const5_out dff_const5.v tb_dff_const5.v
vvp dff_const5_out
gtkwave tb_dff_const5.vcd

text

**Synthesis Commands:**  
read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
read_verilog dff_const5.v
synth -top dff_const5
dfflibmap -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib

text

**Simulation Waveform Screenshot:**  
![Insert simulation waveform image here for dff_const5]

**Synthesis Result Screenshot:**  
![Insert synthesis output/netlist image here for dff_const5]

---

## 6. Sequential Optimization: Unused Output Handling

Unused outputs in sequential circuits can cause unnecessary resource usage or synthesis warnings. Optimizing counters and logic to handle these properly can save area and power.

### Lab: counter_opt.v + tb_counter_opt.v

**Description:**  
[Add description about this counter and how unused outputs are optimized]

**Simulation and Synthesis:**  
(Commands similar to above)

**Simulation Waveform Screenshot:**  
![Insert simulation waveform image here for counter_opt]

**Synthesis Result Screenshot:**  
![Insert synthesis output/netlist image here for counter_opt]

---

### Lab: counter_opt2.v + tb_counter_opt.v

**Description:**  
[Add description about counter_opt2 and its optimization]

**Simulation and Synthesis:**  
(Commands similar to above)

**Simulation Waveform Screenshot:**  
![Insert simulation waveform image here for counter_opt2]

**Synthesis Result Screenshot:**  
![Insert synthesis output/netlist image here for counter_opt2]

---
## 7. Summary

Today we learned how combinational and sequential logic can be optimized through techniques such as constant propagation, Boolean simplification, FSM state reduction, cloning, and retiming. These optimizations lead to better silicon utilization, reduced power consumption, and improved timing performance.

The hands-on labs reinforced these concepts by illustrating how synthesis tools like Yosys identify and eliminate redundant logic, simplify expressions, and optimize modular designs.

---
