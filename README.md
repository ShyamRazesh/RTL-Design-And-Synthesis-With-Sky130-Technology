
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
<img width="993" height="384" alt="image" src="https://github.com/user-attachments/assets/73280d24-0964-481a-bf2f-dea518d49d6e" />

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

















# <h3 id="header-3"> Day 3- Combinational and sequential optimization</h3>	 


# <h4 id="header-4">Day 4- GLS blocking vs non-blocking and synthesis-simulation mismatch</h4>


# <h5 id="header-5">Day 5- Optimization in synthesis</h5>




















   
</div>

