 ![OS](https://img.shields.io/badge/OS-Linux-yellow.svg)
 
# About This Workshop
Workshops provide a structured environment where learning, experimentation, and collaboration intersect. They help individuals of all experience levels — from beginners to professionals — to develop essential practical skills, understand emerging technologies, and connect with a like-minded community. The broader goals include enhancing creativity, boosting confidence, and equipping participants with knowledge that extends beyond traditional classroom or work settings.


## Contents
<div class="toc">
  <ul>
    <li><a href="#header-1">Day 1- Introduction to Verilog design and synthesis</a></li>
</div>  
     
<div class="toc">
  <ul>
    <li><a href="#header-2">Day 2- Timing libs, hierarchical vs flat synthesis and efficent flop coding styles</a></li> 
</div>
     
<div class="toc">
  <ul>
    <li><a href="#header-3">Day 3- Combinational and sequential optimization</a></li>
</div>

<div class="toc">
  <ul>
    <li><a href="#header-4">Day 4- GLS blocking vs non-blocking and synthesis-simulation mismatch</a></li>
</div>

<div class="toc">
  <ul>
    <li><a href="#header-5">Day 5- Optimization in synthesis</a></li>

#
Author: Shyam Razesh

Acknowledgments
-----
RTL Design and Synthesis Program by [Mr. Kunal Ghosh](https://www.linkedin.com/in/kunal-ghosh-vlsisystemdesign-com-28084836) 

Reference
----
https://www.vlsisystemdesign.com/rtl-design-using-verilog-with-sky130-technology/?q=%2Frtl-design-using-verilog-with-sky130-technology%2F&v=a98eef2a3105

https://github.com/kunalg123/sky130RTLDesignAndSynthesisWorkshop

https://github.com/kunalg123/vsdflow

# RTL-Design-And-Synthesis-With-Sky130-Technology
This repository contains material from an RTL design workshop focusing on Verilog and the SKY130 open-source technology. The objective of the program is to develop a strong understanding of writing reliable RTL code that behaves consistently when implemented in silicon. The flow covers several key areas, including creating Verilog designs that follow proper coding practices, verifying functionality through functional simulation, building testbenches to check design correctness, performing logic synthesis, and finally running gate-level simulations on the synthesised netlist to ensure the design works as expected after synthesis.

# Prerequisites
 Basic understanding of digital logic (gates, flip-flops, multiplexers, etc.)
 
 Familiarity with Linux shell commands
 
 A Linux environment (or WSL on Windows/macOS)
# Tools: 
github

iverilog

gtkwave

yosys

# <h1 id="header-1">Day 1- Introduction to Verilog design and synthesis</h1>
# Design
 The Design refers to the actual Verilog code (or set of Verilog modules) that implements the intended functionality according to the specified requirements.
 # Testbench
 A Testbench is a simulation environment used to apply test vectors (stimulus) to the design in order to verify its functionality.

# How the Simulator Works
Open-source tools used:
 
`Icarus Verilog (iverilog) ` – Used to compile and simulate Verilog HDL code.

`GTKWave ` – Used to view waveforms from .vcd (Value Change Dump) files.

<img width="995" height="458" alt="image" src="https://github.com/user-attachments/assets/b2fd4ad6-0c8e-4953-81cd-4f7ce608ce78" />

# Tool Installation

 ```bash
        sudo apt install iverilog
        sudo apt install gtkwave
 ```

# Compile Design and Testbench
  ```bash
          iverilog -o output.out design.v testbench.v
  ```

This creates an executable `output.out `.
Run the simulation:
  ```bash
           ./output.out
  ```
It generates a `wave.vcd` file containing signal transitions.

View Waveform
```bash
       gtkwave wave.vcd
```
# Example: Simulation Waveform for 2:1 MUX

<img width="1029" height="522" alt="image" src="https://github.com/user-attachments/assets/72380670-cf38-4b49-91f6-9d8b3ed3f609" />


# Synthesis Flow
A synthesizer converts RTL Verilog code into a gate-level netlist using cells from a technology library (.lib).
We use `Yosys` as the synthesis tool.
The `.lib` file (Standard Cell Library) contains functional, timing, area and power information of logic gates like AND, OR, MUX, INV, DFF, etc.
The synthesizer maps your logic into these standard cells.

# Timing Constraints
For a digital circuit to work properly, two key timing constraints need to be satisfied :
Setup Time
Hold Time

<img width="983" height="453" alt="image" src="https://github.com/user-attachments/assets/0944957a-0189-4fac-bed3-26ed9d998b2d" />

Slower cells are sometimes needed to fix hold time violations.

# Tool Installation
```bash
       sudo apt install yosys
```
# Commands to Synthesize the Design using Yosys
```bash
read_liberty -lib sky130_fd_sc_hd__tt_025C_1v80.lib
read_verilog ../verilog_files/good_mux.v
synth -top good_mux
abc -liberty sky130_fd_sc_hd__tt_025C_1v80.lib
show
write_verilog -noattr good_mux_netlist.v
```
# Command Summary
` read_liberty -lib `
Loads the Sky130 standard cell library.

` read_verilog `
Loads your RTL Verilog code.

`synth -top good_mux`
Performs synthesis; good_mux is set as the top module.

`abc -liberty`
Maps synthesized logic to standard cells from the Sky130 library.

`show`
Displays a schematic of the synthesized circuit (optional).

`write_verilog -noattr`
Exports the final gate-level netlist without extra attributes.

# Synthesized Schematic: 2:1 MUX

<img width="909" height="304" alt="image" src="https://github.com/user-attachments/assets/21bc1ee6-e0b6-4412-ba27-b1b8fab9eaa7" />


# <h2 id="header-2">Day 2- Timing libs, hierarchical vs flat synthesis and efficent flop coding styles</h2>

# Timing Libraries
The `.lib` timing library describes the functional, timing, power and area information of standard cells used during synthesis.

The timing library used here is `sky130_fd_sc_hd__tt_025C_1v80.lib` .

tt stands for Typical Corner.

**Process corners** describe the range of possible variations in the fabrication process — like how fast or slow transistors switch, depending on tiny variations in doping, dimensions or temperature.

The main standard corners are :

TT (Typical-Typical): Nominal transistor speed.

FF (Fast-Fast): Transistors are faster than nominal.

SS (Slow-Slow): Transistors are slower than nominal.

FS / SF: Mixed — NMOS fast, PMOS slow or vice versa.

# Semiconductor chips are designed and tested to function under various PVT conditions to handle real-world situations.
P — Process , V — Voltage , T — Temperature

**025C** means it’s characterized for 25°C,

**1v80** means a supply voltage of 1.8V.

# Types of Synthesis
# Hierarchical Synthesis
It keeps the module hierarchy intact, just like the RTL design . Each module is synthesized separately and sub-modules are retained as blocks.

# Flattened Synthesis
All modules merged into one netlist for better area and power optimization across modules. It is harder to debug.

flatten is the command to do flat synthesis.

Hierarchical synthesis is done for large, modular SoCs, IP-based flows or when reusing verified blocks.

Flattened synthesis is done for maximum performance/area optimization, for small but speed-critical blocks.

# Example :
# Hierarchical Synthesis of multiple_modules.v 
<img width="646" height="173" alt="image" src="https://github.com/user-attachments/assets/98052a13-4068-45f1-819f-c3807545b7f6" />


# Glitches in combinational circuits :
In combinational logic:

Outputs depend on propagation delays through gates. When inputs change, different paths may have different delays.

This can cause temporary incorrect outputs (glitches or hazards) before the final stable output settles.

When Flip-flops are used between combinational circuits it only capture stable values at edge. Glitches that occur while signals are settling don’t matter .

# Flip-Flop Coding Styles
# Asynchronous Reset D Flip-Flop
```bash
module dff_asyncres (input clk, input async_reset, input d, output reg q);
  always @(posedge clk or posedge async_reset)
    if (async_reset)
      q <= 1'b0;
    else
      q <= d;
endmodule
```
<img width="1023" height="412" alt="image" src="https://github.com/user-attachments/assets/9e0cf646-869c-494e-9f2d-ebf213ed0f51" />

This is a D Flip-Flop with an asynchronous reset.

always @(posedge clk or posedge async_reset) means the always block triggers on either:

the rising edge of the clock (posedge clk), or

the rising edge of the async reset signal (posedge async_reset).

If async_reset is asserted (1), q is immediately reset to 0, regardless of the clock.

If async_reset is not asserted, the flip-flop samples input d on the rising clock edge and stores it in q.

# Synchronous Reset D Flip-Flop
```bash
module dff_syncres (input clk, input sync_reset, input d, output reg q);
  always @(posedge clk)
    if (sync_reset)
      q <= 1'b0;
    else
      q <= d;
endmodule
```
<img width="966" height="210" alt="image" src="https://github.com/user-attachments/assets/86e29916-8559-410f-85d8-fad3d6c773c5" />

This D Flip-Flop has a synchronous reset.

The always block is only sensitive to posedge clk.

On the rising edge of the clock.
If sync_reset is 1, q is cleared to 0.

Otherwise, q takes the value of d.

# Synchronous & Asynchronous Reset D Flip-Flop
```bash
module dff_asyncres_syncres (input clk,input async_reset,input sync_reset,input d,output reg q);
always @(posedge clk or posedge async_reset)
begin
  if (async_reset)
    q <= 1'b0;      
  else if (sync_reset)
    q <= 1'b0;       
  else
    q <= d;         
end
endmodule
```
<img width="1027" height="162" alt="image" src="https://github.com/user-attachments/assets/a4ffe349-99af-4dfe-95d7-58433c63b61b" />

If async_reset is high:

q is immediately reset to 0.
If async_reset is low, but sync_reset is high on a clock edge:

On the next rising edge of clk, q is reset to 0 synchronously.
If both resets are low:

On the next rising clock edge, q takes the value of d.

# <h3 id="header-3"> Day 3- Combinational and sequential optimization</h3>	 

# Design Optimization: Combinational & Sequential Circuits
There are verious techniques to optimize combinational and sequential digital circuits to improve performance, area and power efficiency.

# Constant Propagation
Replaces variables with constant values during synthesis.
Simplifies logic and reduces circuit size.
Improves speed and saves resources.

# State Optimization
Used for FSMs (Finite State Machines).
Merges equivalent states to reduce total states.
Optimizes state encoding for minimal logic.
Applies Boolean logic to minimize equations.

# Cloning
Duplicates a logic cell or block to improve performance.
Balances load, reduces wire delays.
Helps meet timing closure.

# Retiming
Shifts flip-flops in a circuit without changing its logic.
Balances path delays.
Lowers critical path delay to achieve higher clock speed.
Maintains functional equivalence.

# Synthesis Flow Example
```bash
$yosys

yosys> read_liberty -lib ../my_lib/lib/sky130_fd_sc_hd__tt_025C_1v80.lib           

yosys> read_verilog opt_check.v                                                     

yosys> synth -top opt_check                                                         

yosys> opt_clean -purge  -------------------// command to do the optimization 

yosys> abc -liberty ../my_lib/lib/sky130_fd_sc_hd__tt_025C_1v80.lib                    

yosys> show 
```
# Example Labs
# Combinational circuits
```bash
module opt_check (input a , input b , output y);
  assign y = a ? b : 0;
endmodule
```

In this code, the ternary operator (a ? b : 0) is initially synthesized as a multiplexer. However, after optimization, constant propagation simplifies the logic, and the expression is effectively reduced to an **AND gate** (y = a & b). Synthesis Result :

<img width="753" height="177" alt="image" src="https://github.com/user-attachments/assets/b4cecffe-f74a-4c59-bd51-33d9ddfd0972" />

```bash
module opt_check2 (input a , input b , output y);
  assign y = a ? 1 : b;
endmodule
```

here also, the ternary operator (a ? 1 : b) is initially synthesized as a multiplexer. However, after optimization, constant propagation simplifies the logic, and the expression is effectively reduced to an   **OR gate** (y = a | b)

<img width="759" height="161" alt="image" src="https://github.com/user-attachments/assets/5b722af9-071f-4e15-a23a-20ec229c1d34" />


# <h4 id="header-4">Day 4- GLS blocking vs non-blocking and synthesis-simulation mismatch</h4>
# Gate-Level Simulation (GLS)
Gate-Level Simulation (GLS) is an essential step in digital design verification. It ensures that the synthesized netlist works correctly after RTL synthesis.

Why perform GLS?

**Synthesis Validation:** Verifies that synthesis tools correctly translated RTL to netlist.
**Timing Verification:** Checks for timing violations using realistic delays (SDF).
**Testability:** Ensures scan chains and other test structures work post-synthesis.

**After Synthesis:** When RTL is converted into a gate-level netlist.
**Before Physical Design:** To catch issues early.

**Types of GLS**
**Functional GLS:** Logic-only simulation (zero/unit delays).
**Timing GLS:** Uses annotated timing data to check real-world behavior.

**Labs**
**Lab 1: Ternary MUX**

```bash
module ternary_mux (input i0, i1, sel, output y); assign y = sel ? i1 : i0; endmodule
```

```bash
read_liberty sky130_fd_sc_hd__tt_025C_1v80.lib
read_verilog ternary_mux.v
synth -top ternary_mux
abc -liberty sky130_fd_sc_hd__tt_025C_1v80.lib
write_verilog -noattr ternary_operator_mux_netlist.v
iverilog ../my_lib/verilog_modelprimitives.v ../my_lib/verilog_modelsky130_fd_sc_hd.v ternary_mux.v tb_ternary_mux.v
./a.out
gtkwave tb_ternary_mux.vcd
```
**Before synthesis**
<img width="1254" height="516" alt="image" src="https://github.com/user-attachments/assets/c1ecbf94-8a90-433c-8340-73a9552c273d" />

**After Synthesis**
<img width="1258" height="512" alt="image" src="https://github.com/user-attachments/assets/2e99cac0-5f5b-42fe-83fa-9182fd64b49f" />

**Lab 2: Bad MUX**
```bash
module bad_mux (input i0, i1, sel, output reg y); always @(sel) begin // Incomplete sensitivity! if (sel) y <= i1; else y <= i0; end endmodule
```
```bash
read_liberty sky130_fd_sc_hd__tt_025C_1v80.lib
read_verilog bad_mux.v
synth -top bad_mux
abc -liberty sky130_fd_sc_hd__tt_025C_1v80.lib
write_verilog -noattr bad_mux_netlist.v
iverilog ../my_lib/verilog_model/primitives.v ../my_lib/verilog_model/sky130_fd_sc_hd.v bad_mux.v tb_bad_mux.v
./a.out
gtkwave tb_bad_mux.vcd
```
**Before synthesis**
<img width="1246" height="511" alt="image" src="https://github.com/user-attachments/assets/787e098e-6ef0-4ae8-8da4-70330661aba0" />


**After synthesis**
<img width="1244" height="506" alt="image" src="https://github.com/user-attachments/assets/e097508d-452d-4942-aaa2-a838726aba0a" />

**Lab 3: BLocking Caveat**
```bash
module blocking_caveat (input a, b, c, output reg d); reg x; always @(*) begin d = x & c; // Uses OLD x! x = a | b; // Too late end endmodule
```

```bash
read_liberty sky130_fd_sc_hd__tt_025C_1v80.lib
read_verilogblocking_caveat.v
synth -top blocking_caveat
abc -liberty sky130_fd_sc_hd__tt_025C_1v80.lib
write_verilog -noattr blocking_caveat_netlist.v
iverilog ../my_lib/verilog_model/primitives.v ../my_lib/verilog_model/sky130_fd_sc_hd.v blocking_caveat.v tb_blocking_caveat.v
./a.out
gtkwave tb_blocking_caveat.vcd
```
**Before synthesis**
<img width="1250" height="542" alt="image" src="https://github.com/user-attachments/assets/abfa1b44-caf2-4a0d-b830-497affa34a73" />

**After synthesis**
<img width="1251" height="511" alt="image" src="https://github.com/user-attachments/assets/9db6daea-e1fe-4eaa-a60a-ba1b74944699" />


# <h5 id="header-5">Day 5- Optimization in synthesis</h5>

# Conditional Logic
If-else chains implement decision logic inside procedural blocks for behavioral modeling.

# Basic Form
if (test_expr) begin // Execute when true end else if (test_expr2) begin // Alternative condition end else begin // Catch-all case end

Rule: Every output must get assigned regardless of input combinations in combo blocks.

# Unwanted Latch Creation
Latches appear when combo logic (always @(*)) leaves some outputs un-driven in certain conditions. Synthesis tools create memory elements you didn't intend.

**Problem Case**
always @(*) begin if (enable) out = data; // What happens when enable=0? end
**
Proper Fix**
always @(*) begin case(enable) 1'b1: out = data; default: out = 1'b0; // Always assigned endcase end

**Conditional Labs**
**Lab 1: Missing Assignment Path**
module missing_path (input i0, i1, i2, output reg y); always @(*) begin if (i0) y <= i1; // No fallback → latch created end endmodule
<img width="1247" height="512" alt="image" src="https://github.com/user-attachments/assets/3d47ca25-8b92-4141-9d0a-fb7a762405f0" />

**Lab 2: Result After Synthesis**
<img width="762" height="370" alt="image" src="https://github.com/user-attachments/assets/196132fd-aeeb-448f-8612-70d47d9db483" />


**Lab 3: Chained Conditions (Incomplete)**
module chain_missing (input i0,i1,i2,i3, output reg y); always @(*) begin if (i0) y <= i1; else if (i2) y <= i3; // Gap when both false end endmodule

<img width="1244" height="538" alt="image" src="https://github.com/user-attachments/assets/35885531-6d76-4b39-8392-05c6370b99b1" />

**Lab 4: Synthesis Output**

<img width="763" height="205" alt="image" src="https://github.com/user-attachments/assets/78500eaa-5440-487c-8f15-4bd1a697e297" />


**Lab 5: Full Coverage Case** 
module safe_case (input i0,i1,i2, input [1:0] sel, output reg y); always @(*) begin case(sel) 2'b00: y = i0; 2'b01: y = i1; default: y = i2; // Covers all possibilities endcase end endmodule
<img width="1247" height="538" alt="image" src="https://github.com/user-attachments/assets/abbdfcd0-ad5e-498c-9b65-713afa2db7be" />

**Lab 6: Clean Synthesis**

<img width="1260" height="253" alt="image" src="https://github.com/user-attachments/assets/106f68b9-778c-48e3-b3fc-081806c9a2f9" />

**Lab 7: Overlapping Patterns**
module overlap_case (input i0,i1,i2,i3, input [1:0] sel, output reg y); always @(*) begin case(sel) 2'b00: y = i0; 2'b01: y = i1; 2'b10: y = i2; 2'b1?: y = i3; // Wildcard creates priority issues endcase end endmodule
<img width="1261" height="540" alt="image" src="https://github.com/user-attachments/assets/855032e1-e9a5-48e9-a48d-16a5e3a4a644" />

**Lab 8: Selective Assignments**
module selective_assign (input i0,i1,i2, input [1:0] sel, output reg y,x); always @(*) begin case(sel) 2'b00: begin y=i0; x=i2; end 2'b01: y = i1; // x forgotten here! default: begin x=i1; y=i2; end endcase end endmodule
<img width="1260" height="295" alt="image" src="https://github.com/user-attachments/assets/755bbe7e-99ee-47be-b505-c6a2dddd423b" />

**Partial Case**
<img width="1264" height="400" alt="image" src="https://github.com/user-attachments/assets/00993bee-046a-4c53-9649-333455d4356c" />





   
</div>

