# Synopsys Design Compiler: Brief Overview

**Synopsys Design Compiler** is a leading Electronic Design Automation (EDA) software tool used in the semiconductor industry for RTL (Register Transfer Level) synthesis. It translates digital logic designs written in hardware description languages (like Verilog or VHDL) into optimized gate-level representations ready for physical design and fabrication. Design Compiler enables engineers to:

- Convert behavioral or structural code to standard-cell implementations.
- Optimize designs for area, power, and speed.
- Check for and resolve design rule and timing violations.
- Generate standard output files (e.g., gate-level netlists, synthesis reports).

It is especially valued for handling complex ASIC and SoC (System on Chip) designs, supporting multiple technology libraries, and integrating with industry-standard EDA workflows.

# Half-Adder Project with Synopsys Design Compiler

## Overview
This project implements a simple **half-adder** using Verilog HDL and synthesizes it using **Synopsys Design Compiler**. It is intended as a beginner-level digital design project to get familiar with the ASIC synthesis workflow using industry-standard tools.

---
## Features

- Verilog design of a half adder
- Synthesis script in TCL for Synopsys DC
- Basic hands-on with Design Compiler environment
- Tried and tested on Linux systems with csh shell

---

## Getting Started

This guide walks you through setting up your environment and running your half-adder design with Synopsys Design Compiler.

### Prerequisites

Make sure you have the following:

- Access to **Synopsys Design Compiler** (installed via university or organization tools)
- Unix/Linux machine with terminal access
- Shell: **csh**
- Access/setup to the **Cadwork** directory (organization-specific project area)

---

## 🚀 Setup Instructions

### Step 1: Open Synopsys Directory

1. Open your **file manager**.
2. Navigate to: Home > Cadwork > Synopsys

### Step 2: Open Terminal
- Right-click inside the `Synopsys` folder or use system terminal to open a shell window in this directory.

### Step 3: Run Setup Commands
In your terminal, run the following setup commands one by one:
```text
csh
source .cshrc
mkdir 190
cd 190
```
> This changes your shell to C shell, sets up the Synopsys environment, creates a folder `190`, and navigates into it.
---

## File Structure
Your project folder should look like this after adding source files:
```
190/
├── half_adder.v # Verilog HDL for Half Adder
├── synthesize.tcl # TCL script to run synthesis
├── README.md # Project documentation
├── output/ # (Auto-generated synthesis reports, logs)
```

---
**type the command**: gedit ha.v
Type the code in the editor:
## Verilog Code
```
module ha(input a,b, output s,c);
assign s=a^b;
assign c=a&b;
endmodule
```
Similarly: ha_tb.v
```
`timescale 1ns/1ps

module half_adder_tb;
    // Testbench signals
    reg a, b;
    wire s,c;
    // Instantiate the Half Adder module
    ha uut ( .a(a),.b(b), .s(s),.c(c));
    initial begin
        // Display header
        $display("A B | SUM CARRY");
        $display("---------------");

        // Test all possible input combinations
        a = 0; b = 0; #10;
        $display("%b %b |  %b    %b", a, b, sum, carry);

        a = 0; b = 1; #10;
        $display("%b %b |  %b    %b", a, b, sum, carry);

        a = 1; b = 1; #10;
        $display("%b %b |  %b    %b", a, b, sum, carry);

        $finish;
    end
endmodule
```
## Purpose of `.sdc` and `.tcl` Files

### What is a `.sdc` File?

- **Stands for:** Synopsys Design Constraint.
- **Purpose:** Contains timing and physical constraints used by synthesis and place-and-route tools.
- **Content Examples:**
  - Clock definitions and constraints
  - Input/output delay specifications
  - Design environment parameters (e.g., loading, operating conditions)
- **Importance:** Ensures that the synthesized design meets real-world performance requirements by guiding the tool on critical paths, timing budgets, and other physical considerations.

### What is a `.tcl` File?

- **Stands for:** Tool Command Language (pronounced “tickle”).
- **Purpose:** Contains scripts automating tool operations—such as reading design files, invoking synthesis, reporting results, and managing file outputs.
- **Content Examples:**
  - Commands to read HDL files and libraries
  - Flow control and automation (e.g., for batch synthesis, running multiple commands)
  - Report generation (timing, area, power)
  - Custom tool behaviors or batch processes
- **Importance:** Enables reproducible, automated design flows and reduces human error. TCL scripting is widely supported across EDA tools and is a key skill for VLSI/ASIC engineers.

### Summary Table

| File Type  | Full Form                         | Main Purpose                                     | Example Use                           |
|------------|-----------------------------------|--------------------------------------------------|---------------------------------------|
| `.sdc`     | Synopsys Design Constraints       | Specify design timing/physical constraints       | Set clock, IO delay, environment      |
| `.tcl`     | Tool Command Language script      | Automate tool operations and flow control        | Run synthesis, generate reports, etc. |

Both `.sdc` and `.tcl` files are critical for ensuring that the chip design is functionally correct, adheres to timing requirements, and that design flows are efficient and automated.


## To view waveform:

### 1. Compile Design + Testbench with vcs:
```
vcs mux2.v mux2_tb.v -R -debug_all
```
* -R: Runs the simulation after compilation
* -debug_all: Enables all signals for waveform viewing in DVE

### 2. Open Waveforms in GUI:
```
dve -vpd vcdplus.vpd &
```
- DVE will open → Use the GUI to:
 - Add signals from the hierarchy
 - Zoom, pan, and inspect transitions
 - Save waveforms for reports


