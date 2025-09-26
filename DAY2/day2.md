# Day 2: Timing Libraries, Synthesis Strategies, and RTL Optimization

Welcome to Day 2! In this session, we dive deep into the practical aspects of digital design workflows. We will explore the standard cell timing libraries that form the foundation of synthesis, compare different synthesis methodologies, and examine efficient coding styles for fundamental building blocks like flip-flops and simple arithmetic operations.

---

# Table of Contents

- [A Walkthrough of the Liberty Timing Library](#1-a-walkthrough-of-the-liberty-timing-library)
- [Hierarchical vs. Flattened Synthesis](#2-hierarchical-vs-flattened-synthesis)
- [The Role of Flip-Flops in Digital Design](#3-the-role-of-flip-flops-in-digital-design)
- [RTL Coding for Synthesis Optimization](#4-rtl-coding-for-synthesis-optimization)

---

## 1. A Walkthrough of the Liberty Timing Library

The foundation of any synthesis process is the technology library, typically provided in the Liberty (`.lib`) format. This file characterizes the standard cells (like AND, OR, Flip-Flops) available in a specific Process Design Kit (PDK). For our labs, we use `sky130_fd_sc_hd__tt_025C_1v80.lib` from the open-source SkyWater 130nm PDK.

### Decoding the Library Name: PVT Corners

The name `tt_025C_1v80` specifies the **PVT corner** for which this library is characterized. PVT stands for **Process, Voltage, and Temperature**, three key factors that affect a chip's performance. Simulating a design across different PVT corners ensures its robustness.

* **P (Process)**: `tt` stands for **Typical-Typical**. This represents the nominal or expected manufacturing process corner for both NMOS and PMOS transistors. Other common corners include `ff` (Fast-Fast) and `ss` (Slow-Slow).
* **V (Voltage)**: `1v80` indicates a nominal operating voltage of **1.80 Volts**.
* **T (Temperature)**: `025C` represents an operating temperature of **25° Celsius**.

By synthesizing with this library, we are targeting how a typical chip would behave under normal voltage and room temperature conditions.

### Inside the `.lib` File

The Liberty file is a detailed ASCII file containing timing and power information for every standard cell. Let's look at some key sections.

#### Header Information
The header defines global parameters and units for the entire library.

* **`technology(cmos)`**: Specifies the underlying transistor technology.
* **`delay_model: table_lookup`**: This is a common delay model where timing values are stored in multi-dimensional lookup tables (LUTs). The synthesis tool interpolates values from these tables based on factors like input transition time and output load capacitance.
* **Units**: The header defines the units for various physical quantities. For example:
    * `time_unit`: "1ns"
    * `voltage_unit`: "1V"
    * `current_unit`: "1mA"
    * `leakage_power_unit`: "1nW"


#### Standard Cell Characterization
The library contains detailed descriptions for hundreds of cells. Each `cell` block defines a specific logic gate or flip-flop with a particular drive strength (e.g., `sky130_fd_sc_hd__and2_1`, `sky130_fd_sc_hd__and2_2`).

Key attributes for each cell include:
* **`area`**: The physical area the cell occupies on the silicon, crucial for chip floorplanning.
* **`pin` descriptions**: Defines each input and output pin's characteristics, such as direction, capacitance, and timing arcs.
* **Timing and Power LUTs**: These tables define the cell's behavior. For instance, a `cell_delay` table will show the gate's propagation delay based on the input signal's slew rate and the capacitive load on the output.

**Example: Drive Strength**
You will often see the same gate with different suffixes, like `_1`, `_2`, `_4`. These represent different **drive strengths**.
* A **higher drive strength** cell (e.g., `and2_4`) has larger transistors.
* **Advantages**: It can drive larger capacitive loads with less delay.
* **Disadvantages**: It consumes more power and occupies a larger area.

The synthesis tool automatically chooses the cell with the appropriate drive strength to meet the design's timing constraints while minimizing area and power.

<div align="center">
  <img src="https://github.com/ShravanaHS/week1-riscV-soc-tapeout-vsd/blob/main/images/lib1.png"/>
  <br>
  <b>sky130_fd_sc_hd__tt_025C_1v80.lib</b>
</div>

<div align="center">
  <img src="https://github.com/ShravanaHS/week1-riscV-soc-tapeout-vsd/blob/main/images/lib2.png"/>
  <br>
  <b>sky130_fd_sc_hd__tt_025C_1v80.lib</b>
</div>

---

## 2. Hierarchical vs. Flattened Synthesis

Synthesis tools can approach a multi-module RTL design in two primary ways: hierarchical or flattened.

### Hierarchical Synthesis
In this approach, the synthesis tool preserves the module boundaries defined in the RTL code. Each module is synthesized as a distinct entity, and then they are connected together at the top level.

**Why use it?**
* **Divide and Conquer**: For massive System-on-Chip (SoC) designs, synthesizing the entire chip at once is computationally expensive and difficult to manage. Hierarchical synthesis allows teams to work on and synthesize different blocks independently.
* **Reusability**: If a module is instantiated multiple times, it can be synthesized once and its netlist can be reused, saving compilation time.
* **Debugging**: The synthesized netlist maintains a structure that directly corresponds to the original RTL, making it much easier to trace signals and debug issues.

### Flattened Synthesis
In this approach, the synthesis tool dissolves all module boundaries and merges the entire design logic into a single, flat netlist before performing optimization.

**Why use it?**
* **Global Optimization**: By removing module walls, the tool can perform optimizations across the entire design. For example, it can merge logic from two different modules or eliminate redundant logic that spans module boundaries. This can often lead to a better final result in terms of Power, Performance, and Area (PPA).

### Key Differences

| Aspect             | Hierarchical Synthesis                           | Flattened Synthesis                              |
| ------------------ | ------------------------------------------------ | ------------------------------------------------ |
| **Hierarchy** | Preserved; matches RTL structure                 | Collapsed into a single-level netlist            |
| **Optimization** | Performed within each module's boundaries        | Performed globally across the entire design      |
| **Runtime** | Generally faster for very large designs          | Can be very slow for large, complex designs      |
| **Debugging** | Easier, as the netlist maps clearly to the RTL   | More difficult, as original structure is lost    |
| **Use Case** | Large multi-team projects, block-based design    | Smaller designs or when maximum optimization is critical |

<br>
<div align="center">
  <img src="https://github.com/ShravanaHS/week1-riscV-soc-tapeout-vsd/blob/main/images/alusubm.png" />
  <br>
  <b> Hierarchical Synthesis of ALU </b>
</div>
<br>
<div align="center">
  <img src="https://github.com/ShravanaHS/week1-riscV-soc-tapeout-vsd/blob/main/images/netlist.png"  />
  <br>
  <b>Flatten Synthesis of ALU</b>
</div>

---

## 3. The Role of Flip-Flops in Digital Design

Combinational circuits, made of logic gates, suffer from propagation delays. As signals travel through different paths in a netlist, they arrive at the output at slightly different times, which can cause temporary, unwanted signal changes known as **glitches** or **hazards**.

**Flip-flops** are the fundamental memory elements in sequential logic. Their primary role is to act as synchronizing elements. By capturing data only on the active edge of a clock signal, they break long combinational paths and ensure that glitches do not propagate through the system, leading to a stable and predictable design.

### Reset Mechanisms

A reset is crucial for bringing a sequential circuit into a known, predictable state.

#### Asynchronous Reset
This reset is independent of the clock. When the reset signal is asserted, the flip-flop's output is immediately forced to its reset state, regardless of any clock activity.

```verilog
module dff_async_reset (
    input clk, d, async_reset,
    output reg q
);
    // Sensitivity list includes the reset signal
    always @(posedge clk or posedge async_reset) begin
        if (async_reset)
            q <= 1'b0;
        else
            q <= d;
    end
endmodule
```
<div align="center">
  <img src="https://github.com/ShravanaHS/week1-riscV-soc-tapeout-vsd/blob/main/images/asynsim.png" />
  <br>
  <b>Simulation</b>
</div>
<br>
<div align="center">
  <img src="https://github.com/ShravanaHS/week1-riscV-soc-tapeout-vsd/blob/main/images/asynsis.png" />
  <br>
  <b>Synthesis</b>
</div>

#### Synchronous Reset
This reset is synchronized with the clock. The reset action only occurs on the next active clock edge after the reset signal is asserted.

```verilog
module dff_sync_reset (
    input clk, d, sync_reset,
    output reg q
);
    // Sensitivity list only includes the clock
    always @(posedge clk) begin
        if (sync_reset)
            q <= 1'b0;
        else
            q <= d;
    end
endmodule
```
<div align="center">
  <img src="https://github.com/ShravanaHS/week1-riscV-soc-tapeout-vsd/blob/main/images/synsim.png" />
  <br>
  <b>Simulation</b>
</div>
<br>
<div align="center">
  <img src="https://github.com/ShravanaHS/week1-riscV-soc-tapeout-vsd/blob/main/images/synsis.png" />
  <br>
  <b></b>
</div>
<br>
<div align="center">
  <img src="https://github.com/ShravanaHS/week1-riscV-soc-tapeout-vsd/blob/main/images/synsys.png" />
  <br>
  <b>Synthesis</b>
</div>

#### Syncheronous Asynchronous Reset
This type of reset combines the benefits of both synchronous and asynchronous resets, helping in timing control during reset assertion while avoiding clock-related delays during deassertion.

```verilog
module dff_async_sync_reset (
    input  clk,          // clock
    input  d,            // data input
    input  async_reset,  // asynchronous reset (active high)
    input  sync_reset,   // synchronous reset (active high)
    output reg q         // output
);

    // Include async_reset in sensitivity list for immediate reset
    always @(posedge clk or posedge async_reset) begin
        if (async_reset)       // asynchronous reset
            q <= 1'b0;
        else if (sync_reset)   // synchronous reset (checked only on clock edge)
            q <= 1'b0;
        else
            q <= d;            // normal operation
    end

endmodule

```
<div align="center">
  <img src="https://github.com/ShravanaHS/week1-riscV-soc-tapeout-vsd/blob/main/images/synasynsim.png" />
  <br>
  <b>Simulation</b>
</div>
<br>
<div align="center">
  <img src="https://github.com/ShravanaHS/week1-riscV-soc-tapeout-vsd/blob/main/images/synasynsyn.png" />
  <br>
  <b>Synthesis</b>
</div>

#### Simulation of Flip-Flops
You can simulate these behaviors using a testbench and tools like Icarus Verilog (iverilog) for compilation and GTKWave (gtkwave) for waveform viewing to observe how the q output responds to the d, clk, and reset signals.

## 4. RTL Coding for Synthesis Optimization
How you write your RTL code can have a significant impact on the resulting hardware. A smart synthesis tool can often optimize simple arithmetic operations into highly efficient logic, sometimes reducing them to just wire connections.

Example 1: Multiplication by a Power of 2
Consider multiplying a 3-bit number a by 2.
assign y = a * 2;

In binary, multiplying by 2 is equivalent to a left logical shift by one bit. The synthesis tool recognizes this and will not infer a complex multiplier circuit. Instead, it implements the operation with simple wiring.



```Verilog

module mult_by_2 (
    input  [2:0] a,
    output [3:0] y
);
    assign y = a * 2; // Equivalent to y = a << 1;
endmodule
```
Synthesized Logic: The tool maps a[2:0] directly to y[3:1] and ties y[0] to 1'b0 (ground). This is extremely efficient.

<div align="center">
  <img src="https://github.com/ShravanaHS/week1-riscV-soc-tapeout-vsd/blob/main/images/mult2.png" />
  <br>
  <b>Simulation</b>
</div>
<div align="center">
  <img src="https://github.com/ShravanaHS/week1-riscV-soc-tapeout-vsd/blob/main/images/mult2ss.png" />
  <br>
  <b></b>
</div>
<div align="center">
  <img src="https://github.com/ShravanaHS/week1-riscV-soc-tapeout-vsd/blob/main/images/mylt2syn.png" />
  <br>
  <b>Synthesis</b>
</div>



Example 2: Multiplication by a Constant
- Consider multiplying a 3-bit number a by 9.
- assign y = a * 9;
- The synthesis tool can break this down using the distributive property:
- a * 9 = a * (8 + 1) = (a * 8) + (a * 1)
- a * 8 is a simple left shift by 3 bits.
- a * 1 is just a.
The hardware implementation becomes a shifter (wires) and an adder, which is much smaller and faster than a generic multiplier

```verilog
module mult_by_9 (
    input  [2:0] a,
    output [5:0] y
);
    assign y = a * 9;
endmodule
```
Synthesized Logic: The tool creates the two terms (a << 3 and a) and adds them together using an adder cell from the standard cell library.
<div align="center">
  <img src="https://github.com/ShravanaHS/week1-riscV-soc-tapeout-vsd/blob/main/images/mult8syn.png" />
  <br>
  <b>Synthesis</b>
</div>

Writing synthesizable and efficient RTL is a key skill. Understanding how a tool might interpret your code allows you to guide it toward a better hardware implementation.
