# Day 5: Optimization in Verilog Synthesis

 This section thoroughly explains optimization techniques related to `if-else` statements, case statements, partial assignments, and generation constructs in Verilog design. Each topic covers theory with detailed examples and practical implications.

---

## 1. If Statement Optimization

### Priority and Execution

- In synthesis, the first `if` condition has the **highest priority**.
- If multiple conditions are true, the **first matching statement executes** and others are skipped.

**Example:**
```
always @(*) begin
if (a)
y = 1;
else if (b)
y = 2;
else
y = 3;
end
```
Here, if `a` and `b` are both true, `y` is assigned `1` because `if (a)` comes first.

---

### Inferred Latches and Incomplete If Statements

An **incomplete if statement** (no `else`) causes the tool to infer a latch; the output variable holds its previous value if no assignment occurs.

**Example:**
```
always @(*) begin
if (en)
q = d;
// No 'else' for q: latch inferred
end
```
- Fix by assigning `q` in all code paths (using `else` or initial assignment).

---

### Counter Example: Control with Enable and Reset

A common sequential design:
```
always @(posedge clk)
if (rst)
out <= 0;
else if (enable)
out <= out + 1;
```
No else: out holds previous value (like a latch)


---

## 2. Partial Assignments and Case Statement Optimization

### Case (or if-else vs. case)

- Both `if-else` and `case` can infer latches if not all variables assigned in every path.
- **Case statement** acts like a large multiplexer.

**Example:**
```
always @(*) begin
case(sel)
2'b00: y = d0;
2'b01: y = d1;
// Missing default path: y not assigned!
endcase
end
```
Add a `default` to prevent latches.

---

### Partial Assignments in Case

If outputs are only assigned for some cases, latches form for unassigned outputs.
**Best practice:** Assign every output in every case or initialize at block start.

**Example:**
```
always @(*) begin
y = 0; // Initialization avoids latch!
case(sel)
2'b00: y = d0;
2'b01: y = d1;
default: y = d2;
endcase
end
```

---

### If-Else vs. Overlapping Case

- Multiple if statements can be overlapping; ordering determines which executes.
- Cases are mutually exclusive when well written, but wildcards can lead to overlap unless default is used.

---

### Incomplete Case Example and Latch Inference

- **4:1 MUX Example:** A case without a `default` assigns y for two inputs, but for others output is latched.
- Always use full assignment or a default in a case block.

**Example:**
```
always @(*) begin
case(sel)
2'b00: y = i0;
2'b01: y = i1;
// Missing cases for sel=2'b10 and sel=2'b11 cause latches
endcase
end
```

**Solution:** Add `default: y = 1'b0;` or assign y before case statement.

---

## 3. Looping and Generate Constructs in Verilog

---

### 1. For Loops in Verilog

A **for loop** in Verilog is used to streamline repetitive logic, reducing manual coding and making designs more scalable and readable.

- **Synthesizable loops** must have a fixed, compile-time iteration count.
- For loops in always blocks expand into parallel hardware—not sequential code like C.
- Most common usage: implementing bus-wide or multi-bit logic with minimal code.

**Example: Evaluating Expressions**
```
always @(*) begin
for (i = 0; i < 4; i = i + 1)
y[i] = a[i] & b[i];
end
```
This loop builds combinational logic for each bit position.

---

### 2. Use Cases: Different Loops

- **Nested for loops:** Used for multi-dimensional logic or hardware arrays.
- **Single for loop:** Efficient for bus-wide assignments (MUX, DEMUX, adders).
- **Initialization loops:** Initialize arrays or registers in testbenches (not synthesizable in hardware unless used in always/generate blocks).

**Example:**
```
integer i;
always @(*) begin
for (i = 0; i < 8; i = i + 1)
reg_array[i] = in_bus[i];
end
```
This ensures each element is handled identically.

---
#### Lab 1: incomp_if.v

This lab investigates the effect of incomplete if statements on synthesis and simulation in Verilog, highlighting latch inference.

**Verilog Code:**
```
module incomp_if (input i0, input i1, input i2, output reg y);
always @(*) begin
if (i0)
y = i1;
// 'y' not assigned when i0 is 0 → inferred latch
end
endmodule
```

**Simulation (RTL):**
```
iverilog -o incomp_if_sim incomp_if.v tb_incomp_if.v
vvp incomp_if_sim
gtkwave tb_incomp_if.vcd
```

<div align="center">
  <!-- Paste your waveform screenshot here -->
  <br>
  <b>RTL waveform for incomplete if</b>
</div>

**Yosys Synthesis:**
```
read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
read_verilog incomp_if.v
synth -top incomp_if
abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
write_verilog -noattr incomp_if_net.v
quit
```
<div align="center">
  <!-- Paste your synthesized netlist screenshot here -->
  <br>
  <b>Synthesis result for incomplete if (shows latch)</b>
</div>


**GLS:**
```
iverilog -o incomp_if_sim_gls
../my_lib/verilog_model/primitives.v
../my_lib/verilog_model/sky130_fd_sc_hd.v
incomp_if_net.v
tb_incomp_if.v
vvp incomp_if_sim_gls
gtkwave tb_incomp_if.vcd
```

<div align="center">
  <!-- Paste GLS waveform screenshot here -->
  <br>
  <b>GLS waveform for incomplete if</b>
</div>

---

#### Lab 2: incomp_if2.v

Demonstrates nesting and absence of else branches causing multiple latches.

**Verilog Code:**
```
module incomp_if2 (input i0, input i1, input i2, input i3, output reg y);
always @(*) begin
if (i0)
y = i1;
else if (i2)
y = i3;
// no 'else' for y: can infer a latch
end
endmodule
```

**Simulation (RTL):**
```
iverilog -o incomp_if2_sim incomp_if2.v tb_incomp_if2.v
vvp incomp_if2_sim
gtkwave tb_incomp_if2.vcd
```

<div align="center">
  <!-- Paste waveform screenshot here -->
  <br>
  <b>RTL waveform for incomplete nested if</b>
</div>

**Yosys Synthesis:**
```
read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
read_verilog incomp_if2.v
synth -top incomp_if2
abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
write_verilog -noattr incomp_if2_net.v
quit
```

<div align="center">
  <!-- Synthesis output screenshot -->
  <br>
  <b>Synthesis result for incomplete nested if</b>
</div>

**GLS:**
```
iverilog -o incomp_if2_sim_gls
../my_lib/verilog_model/primitives.v
../my_lib/verilog_model/sky130_fd_sc_hd.v
incomp_if2_net.v
tb_incomp_if2.v
vvp incomp_if2_sim_gls
gtkwave tb_incomp_if2.vcd
```
<div align="center">
  <!-- GLS waveform screenshot -->
  <br>
  <b>GLS waveform for incomplete nested if</b>
</div>

---

#### Lab 3: incomp_case.v

Shows latch inference due to missing default in case block.

**Verilog Code:**
```
module incomp_case (input i0, input i1, input i2, input [1:0] sel, output reg y);
always @(*) begin
case (sel)
2'b00 : y = i0;
2'b01 : y = i1;
// Missing 'default': y not assigned for other cases
endcase
end
endmodule
```
**Simulation (RTL):**
```
iverilog -o incomp_case_sim incomp_case.v tb_incomp_case.v
vvp incomp_case_sim
gtkwave tb_incomp_case.vcd
```

<div align="center">
  <!-- Waveform screenshot -->
  <br>
  <b>RTL waveform for incomplete case</b>
</div>

**Yosys Synthesis:**
```
read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
read_verilog incomp_case.v
synth -top incomp_case
abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
write_verilog -noattr incomp_case_net.v
quit
```
<div align="center">
  <!-- Synthesis screenshot -->
  <br>
  <b>Synthesis result for incomplete case</b>
</div>

**GLS:**
```
iverilog -o incomp_case_sim_gls
../my_lib/verilog_model/primitives.v
../my_lib/verilog_model/sky130_fd_sc_hd.v
incomp_case_net.v
tb_incomp_case.v
vvp incomp_case_sim_gls
gtkwave tb_incomp_case.vcd
```
<div align="center">
  <!-- GLS waveform screenshot -->
  <br>
  <b>GLS waveform for incomplete case</b>
</div>

---

#### Lab 4: partial_case_assign.v

Shows partial assignment causing inferred latches on outputs.

**Verilog Code:**
```
module partial_case_assign (
input i0, input i1, input i2,
input [1:0] sel,
output reg y, output reg x
);
always @(*) begin
case(sel)
2'b00: begin
y = i0;
x = i2;
end
2'b01: y = i1;
default: begin
x = i1;
y = i2;
end
endcase
end
endmodule
```

**Simulation (RTL):**
```
iverilog -o partial_case_assign_sim partial_case_assign.v tb_partial_case_assign.v
vvp partial_case_assign_sim
gtkwave tb_partial_case_assign.vcd
```
<div align="center">
  <!-- Waveform screenshot -->
  <br>
  <b>RTL waveform for partial case assignment</b>
</div>

**Yosys Synthesis:**
```
read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
read_verilog partial_case_assign.v
synth -top partial_case_assign
abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
write_verilog -noattr partial_case_assign_net.v
quit
```
<div align="center">
  <!-- Synthesis screenshot -->
  <br>
  <b>Synthesis result for partial case assignment</b>
</div>

**GLS:**
```
iverilog -o partial_case_assign_sim_gls
../my_lib/verilog_model/primitives.v
../my_lib/verilog_model/sky130_fd_sc_hd.v
partial_case_assign_net.v
tb_partial_case_assign.v
vvp partial_case_assign_sim_gls
gtkwave tb_partial_case_assign.vcd
```
<div align="center">
  <!-- GLS waveform screenshot -->
  <br>
  <b>GLS waveform for partial case assignment</b>
</div>

---

#### Lab 5: bad_case.v

Explores incomplete case handling with wildcards, showing risks in real hardware design.

**Verilog Code:**
   ```
module bad_case (
input i0, input i1, input i2, input i3,
input [1:0] sel,
output reg y
);
always @(*) begin
case(sel)
2'b00: y = i0;
2'b01: y = i1;
2'b10: y = i2;
2'b1?: y = i3; // '?' is a wildcard; incomplete cases!
endcase
end
endmodule
```

**Simulation (RTL):**
```
iverilog -o bad_case_sim bad_case.v tb_bad_case.v
vvp bad_case_sim
gtkwave tb_bad_case.vcd
```

<div align="center">
  <!-- Waveform screenshot -->
  <br>
  <b>RTL waveform for bad case handling</b>
</div>

**Yosys Synthesis:**
```
read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
read_verilog bad_case.v
synth -top bad_case
abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
write_verilog -noattr bad_case_net.v
quit
```
<div align="center">
  <!-- Synthesis screenshot -->
  <br>
  <b>Synthesis result for bad case handling</b>
</div>

**GLS:**
```
iverilog -o bad_case_sim_gls
../my_lib/verilog_model/primitives.v
../my_lib/verilog_model/sky130_fd_sc_hd.v
bad_case_net.v
tb_bad_case.v
vvp bad_case_sim_gls
gtkwave tb_bad_case.vcd
```

<div align="center">
  <!-- GLS waveform screenshot -->
  <br>
  <b>GLS waveform for bad case handling</b>
</div>

---

#### Lab 6: comp_case.v

Demonstrates correct case statement coding with full assignment (no latches inferred).

**Verilog Code:**
```
module comp_case (input i0, input i1, input i2, input [1:0] sel, output reg y);
always @(*) begin
case(sel)
2'b00 : y = i0;
2'b01 : y = i1;
default : y = i2;
endcase
end
endmodule
```

**Simulation (RTL):**
```
iverilog -o comp_case_sim comp_case.v tb_comp_case.v
vvp comp_case_sim
gtkwave tb_comp_case.vcd
```

<div align="center">
  <!-- Waveform screenshot -->
  <br>
  <b>RTL waveform for complete case</b>
</div>

**Yosys Synthesis:**
```
read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
read_verilog comp_case.v
synth -top comp_case
abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
write_verilog -noattr comp_case_net.v
quit
```
<div align="center">
  <!-- Synthesis screenshot -->
  <br>
  <b>Synthesis result for complete case</b>
</div>

**GLS:**
```
iverilog -o comp_case_sim_gls
../my_lib/verilog_model/primitives.v
../my_lib/verilog_model/sky130_fd_sc_hd.v
comp_case_net.v
tb_comp_case.v
vvp comp_case_sim_gls
gtkwave tb_comp_case.vcd
```

<div align="center">
  <!-- GLS waveform screenshot -->
  <br>
  <b>GLS waveform for complete case</b>
</div>

### 3. Generate For Loops (Outside Always)

A **generate for loop** is used in the top-level module or within a `generate` block to instantiate hardware elements (e.g., full adders in a ripple-carry adder, gates in a bus).

- Uses the `genvar` keyword for synthesis-recognized iteration.
- Not permitted inside always blocks.
- Expand into actual Verilog code before synthesis—each iteration generates hardware.

**Example:**
```
genvar i;
generate
for (i = 0; i < 4; i = i + 1) begin : gen_loop
and_gate and_inst (.a(in[i]), .b(in[i+1]), .y(out[i]));
end
endgenerate
```
---

### 4. Use Cases of Blocking Statements in Loops

- **Blocking (`=`) assignments** are preferred within always @(*) loops for combinational hardware.
- Ensure correct order of evaluation inside loops when intermediate results are used.

**Example:**
```
always @(*) begin
for (i = 0; i < 4; i = i + 1)
y[i] = x[i] & c; // x[i] must be updated before being used elsewhere
end
```
If assignment order matters, careful loop and blocking usage avoids unintended logic.

---

### 5. MUX and DEMUX Initialization and Implementation Using Loops

- **For loops:** Commonly used to create MUX/DEMUX logic by iterating over inputs, selectors, or outputs.
- **Initialization in always blocks:** Good practice to zero outputs before setting them inside loops; prevents inferred latches.

**MUX Example:**
```
always @(*) begin
y = 1'b0; // Safe default
for (i = 0; i < 4; i = i + 1) begin
if (i == sel)
y = data[i];
end
end
```

**DEMUX Example:**
```
always @(*) begin
y_int = 8'b0;
for (i = 0; i < 8; i = i + 1) begin
if (i == sel)
y_int[i] = in;
end
end
```

---

### 6. Ripple Carry Adder (RCA) with Generate Block

An **RCA** is a classic n-bit adder implemented by chaining single-bit full adders: each carry-out feeds the next carry-in.

- **generate for loop:** Instantiates each full adder dynamically for the required bit width.

**Example (theory):**
```
genvar i;
generate
for (i = 0; i < N; i = i + 1) begin : rca
fa u_fa (.a(A[i]), .b(B[i]), .cin(C[i]), .sum(S[i]), .cout(C[i+1]));
end
endgenerate
```
Where `fa` is a full adder module.

---

### 7. Practical Considerations and Caveats

- **For loops** help scale, but always check that loop bounds are fixed and do not depend on run-time variables.
- **Generate blocks** make hierarchical, parameterized, modular designs possible—especially useful for multipliers, adders, and scalable bus systems.
- Always pre-initialize outputs within always blocks to prevent latches when using loops.
- Blocking assignment within loops must be ordered to match hardware dependencies.

---

*No hardware will execute loops sequentially; all logic resulting from loops is mapped concurrently in the synthesized design. Always confirm loop unrolling and logic connectivity in your synthesis output!*

## Theory Summary and Coding Guidelines

- **Always assign every output in all paths** in combinational blocks to prevent latches.
- Prefer complete `if-else` and populated `case` statements, including defaults.
- Avoid partial assignments and rely on initialization if possible.
- Case statements are like multiplexers; missing arms infer latches.
- Compare code structure and waveform behavior to detect latches and optimize designs.

---
