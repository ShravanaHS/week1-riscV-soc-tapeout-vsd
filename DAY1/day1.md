# Day 1: Introduction to RTL Design, Simulation & Synthesis

# Table of Contents

- [Overview of RTL Design Flow](#1-overview-of-rtl-design-flow)
- [Step-by-Step Simulation Using Icarus Verilog](#4-step-by-step-simulation-using-icarus-verilog)
- [Waveform Visualization Using GTKWave](#5-waveform-visualization-using-gtkwave)
- [Synthesizing Using Yosys](#7-synthesizing-using-yosys)



## 1. Overview of RTL Design Flow

Register-Transfer Level (RTL) design is a critical stage in digital hardware design that describes how data moves between registers and how the logical operations are performed on the data. The process involves writing RTL code in hardware description languages (HDL) such as Verilog or VHDL, simulating the design with given specification to verify functionality, and synthesizing it into a gate-level netlist for hardware realization.


## 2. RTL Design and Simulation Theory

### What is RTL Design?

 RTL Design expresses the functionality of a digital circuit by describing how registers transfer data based on clock cycles and how combinational logic processes this data.
 Written in Verilog or VHDL, RTL is the interface between high-level system design and hardware.

### Simulation in RTL Design

- Simulation uses software tools to mimic and verify digital circuit behavior at the RTL before fabrication.
- It helps identify logical errors or mismatches early.
- Simulators like **Icarus Verilog** compile and run Verilog code and testbenches.
- A **testbench** is a special module that applies inputs (stimulus) to the design and checks outputs.


## 3. Block Diagram for RTL Simulation

## 🔹 Simulator
- A **simulator** is a software tool that executes the **design + testbench** interaction over simulated time.  
- It applies inputs, evaluates RTL behavior, and produces waveforms or logs.  
- Examples: **Icarus Verilog, ModelSim, Synopsys VCS**.  


## 🔹 Design (DUT)
- The **Design Under Test (DUT)** is your RTL code (Verilog/VHDL).  
- It describes the intended hardware functionality using registers and combinational logic.  
- This RTL code is the one later synthesized into hardware gates.  


## 🔹 Testbench
- A **Testbench** is a **non-synthesizable** module that:  
  - Generates input **stimuli** (drives inputs to DUT).  
  - Provides clock & reset.  
  - Observes DUT outputs.  
  - Compares results with expected values.  


## 🔹 Block Diagram 

![Simulation Block Diagram](https://github.com/ShravanaHS/week1-riscV-soc-tapeout-vsd/blob/main/images/rtl.png)

---

## 4. Step-by-Step Simulation Using Icarus Verilog
Simulation is the process of using software tools to mimic the behavior of a digital circuit described in RTL and verify it before implementing it.

### 🔹 Block Diagram 

![Simulation Block Diagram](https://github.com/ShravanaHS/week1-riscV-soc-tapeout-vsd/blob/main/images/simulation.png)

### Verilog codes
- RTL Code
```verilog
module alu(a, b, opcode, result);
    input [7:0] a, b;
    input [3:0] opcode;
    output reg [7:0] result;

    always @(*) begin
        case(opcode)
            4'b0000: result = a + b;       // Addition
            4'b0001: result = a - b;       // Subtraction
            4'b0010: result = a & b;       // AND
            4'b0011: result = a | b;       // OR
            4'b0100: result = a ^ b;       // XOR
            4'b0101: result = ~a;          // NOT a
            4'b0110: result = a << 1;      // Left shift
            4'b0111: result = a >> 1;      // Right shift
            default: result = 8'b00000000; // Default
        endcase
    end
endmodule
```
- Testbench Code
```verilog
`timescale 1ns/1ps

module alu_tb;
    reg [7:0] a, b;
    reg [3:0] opcode;
    wire [7:0] result;

    // Instantiate the ALU
    alu dut (
        .a(a),
        .b(b),
        .opcode(opcode),
        .result(result)
    );

    // Generate VCD file for waveform
    initial begin
        $dumpfile("alu.vcd");
        $dumpvars(0, alu_tb);
    end

    // Test sequence
    initial begin
        // Test Addition
        a = 8'd10; b = 8'd5; opcode = 4'b0000;
        #10;

        // Test Subtraction
        a = 8'd15; b = 8'd7; opcode = 4'b0001;
        #10;

        // Test AND
        a = 8'b10101010; b = 8'b11001100; opcode = 4'b0010;
        #10;

        // Test OR
        a = 8'b10101010; b = 8'b11001100; opcode = 4'b0011;
        #10;

        // Test XOR
        a = 8'b10101010; b = 8'b11001100; opcode = 4'b0100;
        #10;

        // Test NOT
        a = 8'b10101010; b = 8'b00000000; opcode = 4'b0101;
        #10;

        // Test Left Shift
        a = 8'b00001111; b = 8'b00000000; opcode = 4'b0110;
        #10;

        // Test Right Shift
        a = 8'b11110000; b = 8'b00000000; opcode = 4'b0111;
        #10;

        $finish; // End simulation
    end
endmodule
```

### a. Writing Verilog Code and Testbench
Use a text editor such as `gedit`:
```  
gedit alu.v
alu_tb.v
```
![FIles Output](https://github.com/ShravanaHS/week1-riscV-soc-tapeout-vsd/blob/main/images/files.png)

### b. Compile the Code
compile both the codes using iverilog
```bash
iverilog alu.v alu_tb.v
```
or
```bash
iverilog alu.v alu_tb.v -o alu_sim
```

![iverilog Output](https://github.com/ShravanaHS/week1-riscV-soc-tapeout-vsd/blob/main/images/iverilog.png)

This compiles the Verilog code into an executable simulation file `alu_sim`.

### c. Run the Simulation
```bash
./a.out
```
or
```bash
vvp alu_sim
```
Executes the simulation and typically generates a `.vcd` waveform dump if `$dumpfile` and `$dumpvars` are used in the testbench.

---

## 5. Waveform Visualization Using GTKWave
When you run a simulation with **Icarus Verilog**, the **testbench** usually generates a `.vcd` file (Value Change Dump).  
This file stores all signal transitions (0/1/X/Z) during simulation.  
To debug and visualize these signals, we use **GTKWave**.

---

## 🔹 Step 1: Ensure VCD Dump in Testbench

Add the following lines in your testbench:

```verilog
initial begin
  $dumpfile("alu.vcd");   // create waveform file
  $dumpvars(0, alu_tb);   // dump all signals of testbench
end
```
After running the simulation with:
```bash
vvp alu
```
a file `alu.vcd` will be created in your working directory.

## 🔹 Step 2: Open GTKWave
To view the waveform:
```bash
gtkwave alu.vcd
```
👉 If you used a custom name, replace alu.vcd with that filename.

## 🔹 Step 3: Explore GTKWave GUI
When GTKWave opens:
- Left Panel → Shows your design hierarchy (modules, signals).
- Select Signals → Highlight the signals (e.g., A, B, opcode, Result).
- Insert Signals → Click Insert to move them into the waveform viewer.
- Waveform Window → Displays signal transitions over simulation time.
- Zoom/Scroll → Use toolbar buttons or mouse to zoom in/out and navigate.


![GTKWave Output](https://github.com/ShravanaHS/week1-riscV-soc-tapeout-vsd/blob/main/images/wave.png)

---

## 6. Synthesis Theory

### What is Synthesis?

- Synthesis converts RTL descriptions into gate-level logic netlists based on a target technology.
- It produces a netlist showing gates and flip-flops mapped to a standard cell library.
- This gate-level netlist is used for physical implementation on ASIC/FPGA.

### Synthesizer

- A synthesizer is the software, like **Yosys**, that performs this function.
- It requires a **standard cell library (.lib)** containing timings and characteristics of real logic cells.

---

## 7. Synthesizing Using Yosys
![Synthesis Output](https://github.com/ShravanaHS/week1-riscV-soc-tapeout-vsd/blob/main/images/synthesis.png)
### Preparing for Synthesis

- Clone the example repository containing `lib` files and Verilog sources:
```bash

git clone https://github.com/kunalg123/sky130RTLDesignAndSynthesisWorkshop
cd sky130RTLDesignAndSynthesisWorkshop
```
- `lib/sky130_fd_sc_hd__tt_025C_1v80.lib` is the key Liberty file for SkyWater 130nm PDK standard cells.
- `verilog_files/` contains example designs such as `mux.v` and `counter.v`.

### Running Yosys Synthesis Commands

Read the standard cell library
```bash
read_liberty -lib /path/to/sky130_fd_sc_hd__tt_025C_1v80.lib
```

Read the RTL Verilog file
```bash
read_verilog alu.v
```
Set the top-level module
```bash
hierarchy -top alu
```
![FIles Output](https://github.com/ShravanaHS/week1-riscV-soc-tapeout-vsd/blob/main/images/abc.png)
Run synthesis steps
```bash
synth -top alu
abc -liberty /path/to/sky130_fd_sc_hd__tt_025C_1v80.lib
```
![FIles Output](https://github.com/ShravanaHS/week1-riscV-soc-tapeout-vsd/blob/main/images/abc.png)
Write synthesized netlist
```bash
write_verilog -noattr alu_netlist.v
```
Optionally view schematic
```bash
show
```
![yosys Output](https://github.com/ShravanaHS/week1-riscV-soc-tapeout-vsd/blob/main/images/netlist.png)
Exit yosys
```bash
exit
```
---

## 8. Summary

By the end of Day 1, I gained a clear understanding of the RTL design process and its importance in digital hardware development. I learned how to write Verilog RTL code along with a testbench, simulate the design using Icarus Verilog, and generate `.vcd` files for further analysis. I also explored how to visualize and analyze signal transitions and timing behavior using GTKWave, which helps in debugging and validating functionality. In addition, I understood the basics of synthesis using Yosys, including how RTL code is mapped to standard cell libraries to generate a gate-level netlist. Overall, I learned how open-source tools like Icarus Verilog, GTKWave, and Yosys provide a complete RTL-to-netlist flow that can be applied in ASIC and FPGA design.

