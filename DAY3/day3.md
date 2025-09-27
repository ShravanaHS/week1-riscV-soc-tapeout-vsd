
# Day 3: Combinational and Sequential Optimization

Welcome to Day 3 of this workshop! Today we discuss optimization of combinational and sequential circuits, introducing techniques to enhance efficiency and performance.

---
# Table of Contents

- [1. Introduction to Logic Optimization](#1-introduction-to-logic-optimization)
- [2. Combinational Logic Optimization](#2-combinational-logic-optimization)
  - [Lab 1: opt_check1.v](#lab-1-opt_check1v)
  - [Lab 2: opt_check2.v](#lab-2-opt_check2v)
  - [Lab 3: opt_check3.v](#lab-3-opt_check3v)
  - [Lab 4: opt_check4.v](#lab-4-opt_check4v)
  - [Lab 5: multiple_module_opt.v](#lab-5-multiple_module_optv)
- [3. Sequential Logic Optimization](#3-sequential-logic-optimization)
- [5. Labs on Sequential Optimization](#5-labs-on-sequential-optimization)
  - [Lab 1: dff_const1.v](#lab-1-dff_const1v)
  - [Lab 2: dff_const2.v](#lab-2-dff_const2v)
  - [Lab 3: dff_const3.v](#lab-3-dff_const3v)
  - [Lab 4: dff_const4.v](#lab-4-dff_const4v)
  - [Lab 5: dff_const5.v](#lab-5-dff_const5v)
  - [Lab: counter_opt.v + tb_counter_opt.v](#lab-counter_optv--tb_counter_optv)
  - [Lab: counter_opt2.v + tb_counter_opt.v](#lab-counter_opt2v--tb_counter_optv)
- [7. Summary](#7-summary)



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


### Boolean Logic Optimization

Methods such as Karnaugh Maps, Quine-McCluskey, and Boolean algebra simplify logic expressions to their minimal form, eliminating redundant terms.

**Example Boolean Expression Reduction:**  
`assign y = a ? (b ? c : (c ? a : 0)) : (!c)`  
can be reduced by Boolean algebra to a minimal form, reducing gates and delay.

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
```verilog
// Simple redundant OR gate optimization
module opt_check1 (input a, input b, output y);
assign y = a | (a & b);
endmodule
```

**Explanation:**  
Yosys synthesis will simplify the redundant OR-AND combination.

<div align="center">
  <img src="https://github.com/ShravanaHS/week1-riscV-soc-tapeout-vsd/blob/main/images/opt1.png"/>
  <br>
  <b>optcheck1</b>
</div>


---

### Lab 2: opt_check2.v
```verilog
// Redundant NOT logic optimization
module opt_check2 (input a, input b, output y);
assign y = ~(~a & ~b);
endmodule
```

**Explanation:**  
Yosys reduces this to a simplified equivalence of `a | b`.
<div align="center">
  <img src="https://github.com/ShravanaHS/week1-riscV-soc-tapeout-vsd/blob/main/images/opt2.png"/>
  <br>
  <b>optcheck2</b>
</div>

---

### Lab 3: opt_check3.v
```verilog
// Repeated signals for optimization
module opt_check3 (input a, input b, input c, output y);
assign y = (a & b) | (a & b & c);
endmodule
```
**Explanation:**  
Redundant term `(a & b & c)` can be simplified to `(a & b)` only.

<div align="center">
  <img src="https://github.com/ShravanaHS/week1-riscV-soc-tapeout-vsd/blob/main/images/opt3.png"/>
  <br>
  <b>optcheck3</b>
</div>
---

### Lab 4: opt_check4.v
```verilog
// Optimization using distributive law
module opt_check4 (input a, input b, input c, output y);
assign y = (a & b) | (a & c);
endmodule
```

**Explanation:**  
Shows distributive property optimization.

<div align="center">
  <img src="https://github.com/ShravanaHS/week1-riscV-soc-tapeout-vsd/blob/main/images/optcheck4.png"/>
  <br>
  <b>optcheck4</b>
</div>
---

### Lab 5: multiple_module_opt.v
```
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
```

**Explanation:**  
Modular design showcasing synthesis optimization and flattening.

<div align="center">
  <img src="https://github.com/ShravanaHS/week1-riscV-soc-tapeout-vsd/blob/main/images/multimodelopt.png"/>
  <br>
  <b>waveform</b>
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
```
iverilog -o dff_const1_out dff_const1.v tb_dff_const1.v
vvp dff_const1_out
gtkwave tb_dff_const1.vcd
```

**Synthesis Commands:**
```
read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
read_verilog dff_const1.v
synth -top dff_const1
dfflibmap -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
```

**Simulation Waveform Screenshot:**  
<div align="center">
  <img src="https://github.com/ShravanaHS/week1-riscV-soc-tapeout-vsd/blob/main/images/seq1wave.png"/>
  <br>
  <b>waveform</b>
</div>


**Synthesis Result Screenshot:**  
<div align="center">
  <img src="https://github.com/ShravanaHS/week1-riscV-soc-tapeout-vsd/blob/main/images/seq1syn.png"/>
  <br>
  <b>Synthesis</b>
</div>

---

### Lab 2: dff_const2.v

**Description:**  
A D flip-flop that always sets output to 1 regardless of reset.

**Simulation Commands:**
```
iverilog -o dff_const2_out dff_const2.v tb_dff_const2.v
vvp dff_const2_out
gtkwave tb_dff_const2.vcd
```

**Synthesis Commands:**
```
read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
read_verilog dff_const2.v
synth -top dff_const2
dfflibmap -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
```

**Simulation Waveform Screenshot:**  
<div align="center">
  <img src="https://github.com/ShravanaHS/week1-riscV-soc-tapeout-vsd/blob/main/images/seq2wave.png"/>
  <br>
  <b>waveform</b>
</div>

**Synthesis Result Screenshot:**  
<div align="center">
  <img src="https://github.com/ShravanaHS/week1-riscV-soc-tapeout-vsd/blob/main/images/seq2syn.png"/>
  <br>
  <b>Synthesis</b>
</div>

---

### Lab 3: dff_const3.v

**Description:**  
[Add short description for dff_const3]

**Simulation Commands:**
```
iverilog -o dff_const3_out dff_const3.v tb_dff_const3.v
vvp dff_const3_out
gtkwave tb_dff_const3.vcd
```

**Synthesis Commands:**  
```
read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
read_verilog dff_const3.v
synth -top dff_const3
dfflibmap -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib

```

**Simulation Waveform Screenshot:**  
<div align="center">
  <img src="https://github.com/ShravanaHS/week1-riscV-soc-tapeout-vsd/blob/main/images/seq3wave.png"/>
  <br>
  <b>waveform</b>
</div>

**Synthesis Result Screenshot:**  
<div align="center">
  <img src="https://github.com/ShravanaHS/week1-riscV-soc-tapeout-vsd/blob/main/images/seq3syn.png"/>
  <br>
  <b>synthesis</b>
</div>

---

### Lab 4: dff_const4.v

**Description:**  
[Add short description for dff_const4]

**Simulation Commands:**
```
iverilog -o dff_const4_out dff_const4.v tb_dff_const4.v
vvp dff_const4_out
gtkwave tb_dff_const4.vcd
```

**Synthesis Commands:**  
```
read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
read_verilog dff_const4.v
synth -top dff_const4
dfflibmap -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
```

**Simulation Waveform Screenshot:**  
<div align="center">
  <img src="https://github.com/ShravanaHS/week1-riscV-soc-tapeout-vsd/blob/main/images/seq4wave.png"/>
  <br>
  <b>waveform</b>
</div>

**Synthesis Result Screenshot:**  
<div align="center">
  <img src="https://github.com/ShravanaHS/week1-riscV-soc-tapeout-vsd/blob/main/images/multimodopt2.png"/>
  <br>
  <b>owaveform</b>
</div>

---

### Lab 5: dff_const5.v

**Description:**  
[Add short description for dff_const5]

**Simulation Commands:**
```
iverilog -o dff_const5_out dff_const5.v tb_dff_const5.v
vvp dff_const5_out
gtkwave tb_dff_const5.vcd
```

**Synthesis Commands:**  
```
read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
read_verilog dff_const5.v
synth -top dff_const5
dfflibmap -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
```

**Simulation Waveform Screenshot:**  
<div align="center">
  <img src="https://github.com/ShravanaHS/week1-riscV-soc-tapeout-vsd/blob/main/images/sew5wave.png"/>
  <br>
  <b>waveform</b>
</div>

**Synthesis Result Screenshot:**  
<div align="center">
  <img src="https://github.com/ShravanaHS/week1-riscV-soc-tapeout-vsd/blob/main/images/seq5syn.png"/>
  <br>
  <b>synthesis</b>
</div>

---

## 6. Sequential Optimization: Unused Output Handling

Unused outputs in sequential circuits can cause unnecessary resource usage or synthesis warnings. Optimizing counters and logic to handle these properly can save area and power.

### Lab: counter_opt.v + tb_counter_opt.v

**Description:**  
[Add description about this counter and how unused outputs are optimized]

**Simulation and Synthesis:**  
(Commands similar to above)

**Simulation Waveform Screenshot:**  
<div align="center">
  <img src="https://github.com/ShravanaHS/week1-riscV-soc-tapeout-vsd/blob/main/images/ctr1wave.png"/>
  <br>
  <b>owaveform</b>
</div>

**Synthesis Result Screenshot:**  
<div align="center">
  <img src="https://github.com/ShravanaHS/week1-riscV-soc-tapeout-vsd/blob/main/images/ctr1syn.png"/>
  <br>
  <b>synthesis</b>
</div>

---

### Lab: counter_opt2.v + tb_counter_opt.v

**Description:**  
[Add description about counter_opt2 and its optimization]

**Simulation and Synthesis:**  
(Commands similar to above)

**Simulation Waveform Screenshot:**  
<div align="center">
  <img src="https://github.com/ShravanaHS/week1-riscV-soc-tapeout-vsd/blob/main/images/counteropt2.png"/>
  <br>
  <b>counter opt 2</b>
</div>

**Synthesis Result Screenshot:**  
<div align="center">
  <img src="https://github.com/ShravanaHS/week1-riscV-soc-tapeout-vsd/blob/main/images/counterpot2.png"/>
  <br>
  <b>counter opt 2</b>
</div>

---
## 7. Summary

Today we learned how combinational and sequential logic can be optimized through techniques such as constant propagation, Boolean simplification, FSM state reduction, cloning, and retiming. These optimizations lead to better silicon utilization, reduced power consumption, and improved timing performance.

The hands-on labs reinforced these concepts by illustrating how synthesis tools like Yosys identify and eliminate redundant logic, simplify expressions, and optimize modular designs.

---
