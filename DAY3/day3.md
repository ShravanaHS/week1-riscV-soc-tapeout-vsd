
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
  - [Basic Sequential Constant Propagation](#basic-sequential-constant-propagation)
  - [Advanced Sequential Optimizations](#advanced-sequential-optimizations)
    - [State Optimization](#state-optimization)
    - [Cloning](#cloning)
    - [Retiming](#retiming)
- [Labs on Optimization](#4-labs-on-optimization)
  - [Lab 1: opt_check1.v](#lab-1-opt_check1v)
  - [Lab 2: opt_check2.v](#lab-2-opt_check2v)
  - [Lab 3: opt_check3.v](#lab-3-opt_check3v)
  - [Lab 4: opt_check4.v](#lab-4-opt_check4v)
  - [Lab 5: multiple_module_opt.v](#lab-5-multiple_module_optv)
- [Summary](#5-summary)

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

## 5. Summary

Today we learned how combinational and sequential logic can be optimized through techniques such as constant propagation, Boolean simplification, FSM state reduction, cloning, and retiming. These optimizations lead to better silicon utilization, reduced power consumption, and improved timing performance.

The hands-on labs reinforced these concepts by illustrating how synthesis tools like Yosys identify and eliminate redundant logic, simplify expressions, and optimize modular designs.

---
